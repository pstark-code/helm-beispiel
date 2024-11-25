# Helm Beispiel Chart

Dieses Dokument und das dazugehörige Repository soll eine gemütliche Einführung ins Thema "Helm-Chart schreiben" bieten. Wir werden bei weitem nicht alle Features von Helm anschauen, aber wir schauen uns ein paar der wichtigsten Konzepte an.

Eine detailliertere Einführung zum Helm-Chart schreiben findet ihr auf der Helm Website: https://helm.sh/docs/chart_template_guide/

Die Sektionen unten korrespondieren mit den Chart Versionen im `./charts` Ordner. Ihr könnt das "Vergleichen" Feature von Visual Studio Code verwenden, wenn ihr verstehen wollt, wie sich eine bestimmte Datei durch die Versionen verändert hat.


## Helm CLI Benutzen

Eine Helm Chart kann mit dem folgenden Befehl installiert werden:

```
helm install --namespace=beispiel-namespace beispiel-releasename ./charts/tbz-sample-app
```

- Der namespace mit der `--namespace` oder `-n` Option angegeben. 
- Der Releasename wird von Helm verwendet um verschiedene Deployments derselben Chart zu unterscheiden. Er wird auch verwendet um ein Deployment wieder zu entfernen mit `helm uninstall beispiel-releasename`

In den meisten Fällen reichen die Default-Values nicht aus um die Helm-Chart installieren zu können. Es kann mit `--values` oder `-f` ein eigenes Values-File angegeben werden:

```
helm install -n namespace --values=./environments/values-staging.yaml beispiel-releasename ./charts/tbz-sample-app
```

Um ein existierendes Deployment anzupassen, wird `helm upgrade` verwendet. Für eine vereinfachte Benutzung kann `--install` angegeben werden um `helm install` und `helm upgrade` zu kombinieren. In den meisten Fällen ist dies der einzige Befehl der regelmässig eingesetzt wird:

```
helm upgrade --install -n namespace --values=./environments/values-staging.yaml beispiel-releasename ./charts/tbz-sample-app
```

## Eine Eigene Helm Chart Anlegen

Ihr könnt eine der Beispiel Helm-Charts aus diesem Repository kopieren und für euch anpassen oder ihr könnt mit der Helm CLI eine neue Chart generieren:

```
helm create CHART_NAME
```

Während ihr die folgenden Abschnitte durchlest, vergleicht jeweils den Code aus den korrespondierenden Unterordnern im `./charts/` Verzeichnis. Und dann wendet das Gelernte auf eure Helm-Charts an.

Viel Erfolg!

## 1. Vorgegebene Konfigurationsvariablen ("Builtin Objects")

Zur Doku, siehe hier: https://helm.sh/docs/chart_template_guide/builtin_objects/

### 1.1. Release Name

Jedes Helm Deployment hat einen Namen, den "Release Name". Dieser muss auf der Kommandozeile angegeben werden, und ist somit immer gesetzt. Innerhalb der Templates kann so darauf zugegriffen werden:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-tbz-sample-app-web
spec:
[...]
```

Unter dem Objekt `.Release` sind auch noch andere Werte verfügbar, z.B der Namespace. Eine vollständige Liste kann in der Dokumentation gefunden werden: https://helm.sh/docs/chart_template_guide/builtin_objects/

Im Objekt `.Values` sind die Variablen aus den verschiedenen Values-Files verfügbar. Diese werden alle übereinander gelegt.

Sagen wir mal, wir hätten dieses `values.yaml`:

```yaml
# values.yaml
myVariable: "foo"
```
Dann könnten wir auf die Variable `myVariable` danach so zugreifen: `.Values.myVariable`

Im ersten grossen Schritt ersetzen wir jetzt alle Stellen in den YAML Files wo "staging" steht, mit `{{ .Release.Name }}`

### 1.2. Chart Name

Im Gegensatz zum "Release Name", der für jedes Deployment gesetzt werden muss, ist der "Chart Name" festgesetzt pro Helm-Chart. Er wird in der Datei `Chart.yaml` gesetzt.

Wir können mit `.Chart.Name` darauf zugreifen. Es kann zum Beispiel Sinn machen, dies in den Kubernetes-Objektnamen einzusetzen. Ein paar Beispiele:

```
name: {{ .Release.Name }}-tbz-sample-app-web-tls
name: {{ .Release.Name }}-tbz-sample-app-db
name: {{ .Release.Name }}-tbz-sample-app-web
name: {{ .Release.Name }}-tbz-sample-app
```

werden zu:

```
{{ .Chart.Name }}
name: {{ .Release.Name }}-{{ .Chart.Name }}-web-tls
name: {{ .Release.Name }}-{{ .Chart.Name }}-db
name: {{ .Release.Name }}-{{ .Chart.Name }}-web
name: {{ .Release.Name }}-{{ .Chart.Name }}
```

Der Vorteil hier ist, dass wenn ihr den Name ändern müsstet, wird alles gleichzeitig mitgeändert. 

Kleine Nebenbemerkung: Wenn ihr eure eigene Chart anlegt mit `helm init`, dann wird eine Datei namens `_helpers.tpl` angelegt, die ein kleines Hilfstemplate beinhaltet, womit der Chartname bei Bedarf überschrieben werden kann. Wie ihr das verwendet, findet ihr in den anderen Dateien die generiert wurden. 


## 2. Eigene Variablen (values.yaml)

Damit wir unseren Code mehrfach verwenden können, müssen wir noch mehr Sachen konfigurierbar machen. 

Unsere eigenen Variablen werden im `.Values` Objekt aufbewahrt. Die Defaults dafür werden entsprechend in der `values.yaml` Datei eingetragen. Es ist üblich (und sinnvoll!), dort jeweils Kommentare dazu zu schreiben, damit ihr in 3 Monaten (oder 4 Jahren) noch wisst was die ganzen Parameter bedeuten.

Es ist euch freigestellt, wie ihr eure Variablen nennt und verschachtelt. Ich kann euch aber ein paar Tipps mitgeben, die hoffentlich helfen bei der Entscheidungsfindung.

Identifiziert benennbare Gruppierungen in eurem Deployment. Hier in diesem Beispiel wären das "web", darunter fällt ein Deployment, ein Service und ein Ingress und "db", das beinhaltet ein Deployment und ein Service. Diese Namen verwenden wir um ein paar Variablen zu gruppieren.

Beispiel:

```yaml
web:
  enabled: true
  replicaCount: 1
  image:
    repository: ghcr.io/shyim/adminerevo
    tag: "latest"

db:
  enabled: true
  replicaCount: 1
  image:
    repository: docker.io/library/postgres
    tag: "latest"

```

Um diese Werte innerhalb der Templates einzufügen, werden Templatetags genau wie im ersten Teil verwendet. Aber statt `.Chart` oder `.Release` wird `.Values` verwendet. Um den Replicacount aus dem Beispiel zu verwenden fügen wir diesen Tag ins Template ein: `{{ .Values.web.replicaCount }}`

Innerhalb der obigen Gruppierungen können weitere Untergruppen erstellt werden, zum Beispiel `deployment:` und `service:` für Variablen, die spezifisch zu diesen Objekttypen sind. Das obige Beispiel könnten wir erweitern zu:

```yaml
web:
  enabled: true
  deployment: 
    replicaCount: 1
    image:
      repository: "ghcr.io/shyim/adminerevo"
      tag: "latest"
  service: 
    portNumber: 8000
    type: "ClusterIP" 

db:
  enabled: true
  deployment:
    replicaCount: 1
    image:
      repository: docker.io/library/postgres
      tag: "latest"
  service: 
    portNumber: 5432
    type: "ClusterIP" 
```

Häufig muss eine Variable an mehreren Orten verwendet werden. Aber meistens ist es trotzdem ziemlich klar in welcher Gruppe sie hingehört. In unserem Beispiel muss die Variable `.Values.db.service.portNumber` auch im Deployment verwendet werden, damit die Applikation erfolgreich mit der Datenbank kommunizieren kann. 

```yaml
web:
  enabled: true
  database:
    name: "example"
    user: "username"
    password: "password"
    host: "example-service"
    driver: "pgsql"
  deployment: 
    replicaCount: 1
    image:
      repository: "ghcr.io/shyim/adminerevo"
      tag: "latest"
  service: 
    portNumber: 8000
    type: "ClusterIP" 

db:
  enabled: true
  deployment:
    replicaCount: 1
    image:
      repository: docker.io/library/postgres
      tag: "latest"
  service: 
    portNumber: 5432
    type: "ClusterIP" 
```

Hier wird zum Beispiel die Variable `.Values.db.service.portNumber` sowohl in der Datei `service-db.yaml` als auch in `deployment-web.yaml` verwendet.

## 3. Templates Ein- und Ausschalten

Wir können jetzt Text und Zahlen in unsere Templates einfügen lassen. Aber manchmal müssen wir ganze Abschnitte, oder sogar ganze Manifests/Kubernetesobjekte weglassen. Oder ein Objekt mehrfach erstellen mit verschiedenen Werten.

Das können wir mit flow-control Konstrukten machen, wie if/else und loops. (siehe Doku: https://helm.sh/docs/chart_template_guide/control_structures/)

If-else:

```handlebars
{{ if .Values.example.enabled }}
# Do something
{{ else if .Values.otherExample.enabled }}
# Do something else
{{ else }}
# Default case
{{ end }}
```
Die `else if` und die `else` Sektionen können auch weggelassen werden. 

```handlebars
{{ if .Values.example.enabled }}
# Deployment manifest für "example"
{{ end }}
```

Falls ihr loops braucht, schaut euch die Dokumentation an, [das Beispiel](https://helm.sh/docs/chart_template_guide/control_structures/#looping-with-the-range-action) ist sehr gut. 

In unserem Beispiel machen wir die Datenbank optional, falls eine externe Datenbank verwendet wird.

In unserem Fall verwenden wir die Variablen `.Values.db.enabled` und `.Values.web.ingress.enabled` in Deployments, Services und dem Ingresses. 

## Abschliessendes

Jetzt haben wir eine funktionierende, wenn auch einfache Helm-Chart. Am meisten lernt ihr, wenn ihr einfach anfängt, damit zu arbeiten. Benutzt die Gelegenheit, eure eigenen Entscheidungen zu treffen und herauszufinden was euer Stil ist. 

Wenn ihr eine längere Einführung ins Helm-Chart schreiben wollt, kann ich den offiziellen Getting Started Guide empfehlen: 

- https://helm.sh/docs/chart_template_guide/
- bzw: https://helm.sh/docs/chart_template_guide/getting_started/

