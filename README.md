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
- https://github.com/CyberT33N/kubectl-cheat-sheet/blob/main/README.md














































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
<br><br>
__________________________________________________
__________________________________________________
<br><br>
<br><br>

# Nodeport

Vorteile
- **Einfachheit:** Einfach einzurichten und zu verwenden. Ideal für schnelle Tests und lokale Entwicklungsumgebungen.
- **Direkter Zugriff:** Ermöglicht den direkten Zugriff auf den Service über die Minikube-IP und einen spezifischen Port.

Nachteile
- **Portbeschränkung:** Ports sind auf den Bereich 30000-32767 beschränkt, was die Konfiguration einschränken kann.
- **Nicht skalierbar:** Nicht ideal für komplexere Setups oder wenn mehrere Services über unterschiedliche Ports verfügbar gemacht werden müssen.
- **Sicherheit:** Direkter Zugriff auf die Services kann weniger sicher sein als über einen Ingress-Controller.

Empfehlung
- **Für einfache lokale Entwicklung und Tests:** NodePort ist oft ausreichend und einfacher einzurichten.
- **Für komplexere Anwendungen oder wenn du verschiedene Services über unterschiedliche Pfade/Hostnamen erreichbar machen willst:** Ingress ist die bessere Wahl.


- If you create an POD and you want to assign a NodePort to it then you have to create a service for it:
```yaml
# Deploys a new MinIO Pod into the metadata.namespace Kubernetes namespace
#
# The `spec.containers[0].args` contains the command run on the pod
# The `/data` directory corresponds to the `spec.containers[0].volumeMounts[0].mountPath`
# That mount path corresponds to a Kubernetes Persistent Volume Claim
# 
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: minio
  name: minio
  namespace: minio-dev # Change this value to match the namespace metadata.name
spec:
  containers:
  - name: minio
    image: quay.io/minio/minio:latest
    command:
    - /bin/bash
    - -c
    args: 
    - minio server /data --console-address :9001
    volumeMounts:
    - mountPath: /data
      name: minio-storage
  nodeSelector:
    kubernetes.io/hostname: minikube # Correct node hostname
  volumes:
  - name: minio-storage
    persistentVolumeClaim:
      claimName: minio-pvc # Refer to the PVC created above
---
apiVersion: v1
kind: Service
metadata:
  name: minio-service
  namespace: minio-dev
spec:
  type: NodePort
  selector:
    app: minio
  ports:
  - name: api-port      # Name des API-Ports
    protocol: TCP
    port: 9000         # Der Port, auf dem der Service intern läuft (API-Port)
    targetPort: 9000   # Der Port des Containers, auf den der Service weiterleitet
    nodePort: 30000    # Der NodePort, der auf jedem Knoten verfügbar sein wird
  - name: webui-port    # Name des WebUI-Ports
    protocol: TCP
    port: 9001         # Der Port für die WebUI
    targetPort: 9001   # Der Port des Containers, auf den der Service weiterleitet
    nodePort: 30001    # Der NodePort für die WebUI
```
- You can define multiple NodePort at a single service but you must give them a name














































<br><br>
<br><br>
__________________________________________________
__________________________________________________
<br><br>
<br><br>

# Ingress

Vorteile
- **Flexibilität:** Ermöglicht das Routing basierend auf Hostnamen und Pfaden, was besonders nützlich ist für komplexere Anwendungen.
- **TLS-Unterstützung:** Einfachere Konfiguration von HTTPS-Verbindungen.
- **Skalierbarkeit:** Besser geeignet für größere und komplexere Setups, insbesondere wenn viele Services über denselben Zugangspunkt verfügbar gemacht werden sollen.

Nachteile
- **Komplexität:** Erfordert zusätzliche Konfiguration und ein Ingress-Controller muss installiert und konfiguriert sein.
- **Overhead:** Fügt einen zusätzlichen Abstraktionslayer hinzu, was für einfache, lokale Tests möglicherweise unnötig ist.

Empfehlung
- **Für einfache lokale Entwicklung und Tests:** NodePort ist oft ausreichend und einfacher einzurichten.
- **Für komplexere Anwendungen oder wenn du verschiedene Services über unterschiedliche Pfade/Hostnamen erreichbar machen willst:** Ingress ist die bessere Wahl.


































<br><br>
<br><br>
__________________________________________________
__________________________________________________
<br><br>
<br><br>




# minikube
- https://github.com/CyberT33N/minikube-cheat-sheet
