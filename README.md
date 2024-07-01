# Kubernetes Cheat Sheet
Kubernetes Cheat Sheet with the most needed stuff..





# Glossar



## Pod (:pod) (https://kubernetes.io/de/docs/concepts/workloads/pods/)
- Pods sind die kleinsten einsetzbaren Einheiten, die in Kubernetes erstellt und verwaltet werden können.
  - Ein Pod ist eine Gruppe von einem oder mehreren Containern mit gemeinsam genutzten Speicher- und Netzwerkressourcen und einer Spezifikation für die Ausführung der Container.

## Namespace (:ns) (https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
- Organisatorisch
  - Secrets usw. sind nur innerhalb von dem jeweiligen Namespace verfügbar.

## Job (:job) (https://kubernetes.io/docs/concepts/workloads/controllers/job/)
- Einzelne Ausführung im Gegensatz zu Cronjobs.
- Eine Maximale Laufzeit wird definiert danach wird er gelöscht egal ob erfolgreich oder nicht.

## Cronjob (:cj | :cronjob) (https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)
- Wiederholende Ausführung.
- suspend = false wird durch Kubernetes getriggert.

## Deployment (:dp | :deployment) (https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- Sorgen dafür das Pods gestartet werden.
  - Wenn pod fehlschlägt wird ein neuer gestartet. Primär Unterschied zu einem Job.

## Nodes (:nodes) (https://kubernetes.io/docs/concepts/architecture/nodes/)
- Knoten im Kubernetes Cluster.
- Physikalische Maschinen.
  - Master merkt sich pod configs, statefulsets usw.
    - Die anderen worker sorgen nur dafür, dass es ausgeführt wird.

## Replicaset (:replicaset) (https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
- Deployment erstellt Replicaset.
  - Replicaset erstellt Pod.

## Statefulset (:sts | :statefulset) (https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- Hier ist der Name nicht UUID sondern anders als beim Deployment haben wir eindeutige namen mit 0, 1, 2.
  - Pod wird beendet und gestoppt und dann erst neu gestartet.

- Haben nicht nur ein Pod Template sondern auch Template für Volume.
  - Für jeden pod den ich erzeuge kann ich ein PersistentVolume einbinden.

- Wird z.b. für Datenbanken verwendet damit die Daten beim Neustart erhalten bleiben.

## PersistentVolumeClaim (:pvc | :persistentvolumeclaim) (https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- Wenn etwas gespeichert wird dann in PersistentVolume.
  - Die PersistentVolumeClaim sind einem Namespace zugeordnet.

## PersistentVolume (:pv | :persistentvolume) (https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- Namespace unabhängig.

## Secret (:secrets) (https://kubernetes.io/docs/concepts/configuration/secret/)
- Passwörter sollten da drin sein und nicht in configmap oder Umgebungsvariabeln.
  - Projektspezifische Daten würden hier nicht hingehören! Nur allgemeine Daten.
- Funktioniert auch per Namespace.
- helm-charts werden konfiguriert und legen Secret an.

## Configmap (:configmap) (https://kubernetes.io/docs/concepts/configuration/configmap/)
- Hier sind Configs

## Service (:svc | :service) (https://kubernetes.io/docs/concepts/services-networking/service/)
- Kann man wie Domain sehen.
- Wenn sich Anwendungen untereinander unterhalten rufen sie nicht die Pods auf sondern benutzen Services.
- Funktioniert auch per Namespace.
- Man kann mehrere Services erstellen die an die gleichen Pods gehen.


































<br><br>
<br><br>

______________________________________
______________________________________

<br><br>
<br><br>





# kubectl

<details><summary>Click to expand..</summary>




<br><br>
<br><br>

## Logs

<br><br>

## Get previous logs of pod
```shell
kubectl logs mongodb-data-0 -n test --previous
```








<br><br>
<br><br>


## Check current resource usage
```shell
kubectl top nodes
```






<br><br>
<br><br>
____________________________________________________________
____________________________________________________________
<br><br>
<br><br>


## Events

<br><br>

### Check event logs
```shell
kubectl get events -n test --sort-by='.metadata.creationTimestamp'
```









<br><br>
<br><br>
____________________________________________________________
____________________________________________________________
<br><br>
<br><br>

## Port Forward
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/#updating-resources
```shell
kubectl port-forward -n "green" deployment/test-manager 9211:80
kubectl port-forward -n "green" service/bitnami-elasticsearch 1335:9200
```

## Kill Port Forward
```shell
sudo kill -9 $(sudo lsof -t -i:9211) & sudo kill -9 $(sudo lsof -t -i:1335)
```






<br><br>
<br><br>
____________________________________________________________
____________________________________________________________
<br><br>
<br><br>

## Pod
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/

<br><br>

### Get Pod Names
```shell
kubectl get pod -n myNamespace
```

<br><br>

### Kill pod
```shell
kubectl delete pod your-pod-xxxxxx-xxxx6 -n myNamespace
```








<br><br>
<br><br>
____________________________________________________________
____________________________________________________________
<br><br>
<br><br>

## Deployment

<br><br>

### Restart deployment
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/#updating-resources
```shell
kubectl rollout restart deployment deploymentName -n myNamespace
```

<br><br>
<br><br>








<br><br>
<br><br>

## change namespace
```bash
kubectl config set-context --current --namespace=namespaceNameHere
```

## change context
```bash
# Show available contexts
kubectl config get-contexts

# Switch context
kubectl config use contextNameHere
```
























<br><br>
<br><br>
____________________________________________________________
____________________________________________________________
<br><br>
<br><br>

# Job

<br><br>
 
### Create job
```bash
kubectl config use-context minikube
kubectl apply -f test/job.yaml -n namespaceNameHere
```
-f means file
- **In order to re-apply the job you must delete it before


<br><br>
<br><br>

 ### Delete job
 ```bash
kubectl config use-context minikube
kubectl delete -f test/job.yaml -n namespaceNameHere
```
-f means file




 
<br><br>
<br><br>
____________________________________________________________
____________________________________________________________
<br><br>
<br><br>

# Cronjob
 
### Create Cronjob
```bash
kubectl apply -f test/cronjob.yaml -n namespaceNameHere
```

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: mongodb-connection-test
  labels:
    group: test
spec:
  suspend: false
  schedule: "* * * * *"
  jobTemplate:
    metadata:
      name: mongodb-connection-test
      labels:
        group: test
    spec:
      template:
        spec:
          containers:
          - image: nexus.test.local:5000/test:53e89b8
            name: mongodb-connection-test
            env:
            - name: MONGO_OPTION_useNewUrlParser
              value: 'true'
            - name: MONGO_OPTION_useUnifiedTopology
              value: 'true'
            - name: MONGODB_USERNAME
              value: root
            - name: MONGODB_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: mongodb-root-password
                  name: mongodb
            - name: AUTHENTICATION_DATABASE
              value: admin
            - name: MONGODB_ADDRESS
              value: 'mongodb://${MONGODB_USERNAME}:${MONGODB_PASSWORD}@mongodb:27017/test?authSource=${AUTHENTICATION_DATABASE}'
            resources:
              limits:
                cpu: 100m
                memory: 100Mi
              requests:
                cpu: 50m
                memory: 50Mi
          restartPolicy: OnFailure
 ```
 
 
 














 
<br><br>
<br><br>
___________________________________________
___________________________________________

<br><br>
<br><br>
 
## Pod

<br><br>

#### Create Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: "Hello from the environment"
    - name: DEMO_FAREWELL
      value: "Such a sweet sorrow"
```
```bash
kubectl apply -f https://k8s.io/examples/pods/inject/envars.yaml
```







<br><br>
<br><br>
___________________________________________
___________________________________________

<br><br>
<br><br>

## Logs

<br><br>

#### Get logs from pod
```bash
kubectl -n green logs sample-1234abcd
```

<br><br>

#### Get logs from pod with multiple container
```bash
kubectl -n green logs sample-1234abcd -c sample
```

</details> 















































<br><br>
__________________________________________________
__________________________________________________
<br><br>

# Pods




## Restart Policy
- https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy
```
Always: Automatically restarts the container after any termination.
OnFailure: Only restarts the container if it exits with an error (non-zero exit status).
Never: Does not automatically restart the terminated container.

```












<br><br>
<br><br>

## Init Container
- **Will be executed everytime you start your deployment. If you want to execute something before you deploy than you better choose a pre-deploy Hook**
- https://kubernetes.io/docs/concepts/workloads/pods/init-containers/
- deployment.yaml
- In diesem Beispiel benutzen wir abc-test als Basis Image für den Container. Der Init Container wird auf dem Deployment abc-backend angewendet
- You can use sleep the keep the container active to use shell later
```yaml
  initContainers:
  - name: migrations-helper
    resources:
      requests:
        memory: "50Mi"
        cpu: 25m
      limits:
        memory: "100Mi"
        cpu: 100m
    image: '{{ .Values.docker.registry }}/{{ .Values.abc_test.name }}:{{ .Values.abc_test.version }}'
    command:
      - sh
      - -c
      - |
        # ==== DEPENDENCIES ====
        echo "[INIT CONTAINER] Install bash.."
        apk add --no-cache bash

        # ==== VALIDATE ====
        echo "[INIT CONTAINER] Validating script init.."
        # ..

        sleep 6000
  containers:
    # backend (abc-backend)
    - name: {{ .Values.abc_backend.name }}
    # ....
```


<br><br>
<br><br>

### Multiple Init Container
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  initContainers:
  - name: init-container-1
    image: busybox
    command: ['sh', '-c', 'echo Init Container 1; sleep 5']
  - name: init-container-2
    image: busybox
    command: ['sh', '-c', 'echo Init Container 2; sleep 5']
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Main Container; sleep 3600']
```

```shell
kubectl config use-context minikube
kubectl apply -f pod.yaml
```
























<br><br>
__________________________________________________
__________________________________________________
<br><br>





## Container

<br><br>

#### livenessProbe / readinessProbe
```yaml
livenessProbe:
  initialDelaySeconds: 120
  httpGet:
    path: /health
    port: backend
    httpHeaders:
      - name: "crazy-header"
        value: "123"
readinessProbe:
  initialDelaySeconds: 60
  httpGet:
    path: /health
    port: backend
    httpHeaders:
      - name: "crazy-header"
        value: "123"
```






























<br><br>
<br><br>

______________________________________
______________________________________

<br><br>
<br><br>


# Helm
- https://github.com/CyberT33N/helm-cheat-sheet



















<br><br>
__________________________________________________
__________________________________________________
<br><br>





## minikube
- The context for minikube will be created automatically. If you accedentily deleted it you can just restart minikube to create the minikube context again



<br><br>

### Installation
- https://kubernetes.io/de/docs/tasks/tools/install-minikube/#linux
```yaml
# Sie können Minikube unter Linux installieren, indem Sie eine statische Binärdatei herunterladen:
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
  
# So fügen Sie die Minikube-Programmdatei auf einfache Weise Ihrem Pfad hinzu:
sudo cp minikube /usr/local/bin && rm minikube
```

<br><br>
<br><br>

### Deinstallation
```shell
minikube delete
```
