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

<br><br><br><br>


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



<br><br>
__________________________________________________________
__________________________________________________________
<br><br>




# k9s


<br><br>

## statefulset
```bash
k9s
:sts
```

<br><br>

## secrets
```bash
k9s
:secrets
```

<br><br>
<br><br>



## Deploy

<br><br>

#### See current deployments
```bash
k9s
:deploy
```

#### Restart deployment
```bash
k9s
:deploy

# Now choose your deployment and press s. Now you can set the replica to 0 and it will shutdown. Then after this set it again to the amount of replicas which it has before. Then it will restart.
```





<br><br><br><br>

## Logs

<br><br>

#### Logs not showing in Pod
- You can press 0 or:
```bash
~/.k9s/config.yml
k9s.logger.sinceSeconds=-1
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

