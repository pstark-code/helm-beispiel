# Helm Beispiel Chart

## Usage

Die Helm Chart kann mit dem folgenden Befehl installiert werden:

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

## 1. Vorgegebene Konfigurationsvariablen ("Builtin Objects")

Zur Doku, siehe hier: https://helm.sh/docs/chart_template_guide/builtin_objects/

## 1.1. Release Name

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

## 1.2. Chart Name

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


