# Charts HELM

On retrouve :

- un dossier ``charts`` qui contient d’éventuels charts dépendants
- un dossier ``templates`` qui contient les manifests Kubernetes qui seront générés
- un fichier ``Chart.yaml`` qui contient les metada du chart : nom, description,type, version du chart
- un fichier ``values.yaml`` qui contient les valeurs qui seront utilisé pour construire les manifest kubernetes finaux à partir des templates.
- un fichier ``.helmignore`` qui indiquera quels fichiers ou répertoires à ignore lors de la construction de l’artefact. Le pacakge qui sera envoyé dans votre repository helm.

## Installation d'un Chart Helm

- Création d'un namespace de déploiement du Chart

```sh
kubectl create ns adieng
kubectl config set-context --current --namespace=adieng
```

- Verif:

```sh
kubectl.exe config view | yq e .contexts[0].context.namespace
adieng
```
- Installation du Chart my-chart

```sh
λ helm install my-chart .
NAME: my-chart
LAST DEPLOYED: Fri Aug 25 11:09:00 2023
NAMESPACE: adieng
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace adieng -l "app.kubernetes.io/name=my-chart,app.kubernetes.io/instance=my-chart" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace adieng $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace adieng port-forward $POD_NAME 8080:$CONTAINER_PORT
```

- Vérification de ce qui a été crée :

```sh
λ kubectl.exe get all
NAME                            READY   STATUS    RESTARTS   AGE
pod/my-chart-864d4b8ddc-79tmw   1/1     Running   0          9s

NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/my-chart   ClusterIP   100.64.232.223   <none>        80/TCP    9s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-chart   1/1     1            1           10s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/my-chart-864d4b8ddc   1         1         1       10s
```

> On a donc un déployment utilisant un pod exposé avec un service de type ClusterIP.


## Prise en main du langage de templating Go

Helm est écrit en Go. Il utilise le système de templating Go. Il s'inspire au templating Jinja.

- Afficher le contenu du fichier ``templates/service.yaml`` :

```sh
apiVersion: v1
kind: Service
metadata:
  name: {{ include "my-chart.fullname" . }}
  labels:
    {{- include "my-chart.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "my-chart.selectorLabels" . | nindent 4 }}
```
