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




