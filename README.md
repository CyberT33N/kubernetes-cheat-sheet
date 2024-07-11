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
- The context for minikube will be created automatically. If you accedentily deleted it you can just restart minikube to create the minikube context again

<br><br>
<br><br>

## Guide
- https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download

<br><br>

## Installation
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

## Deinstallation
```shell
minikube delete
```









<br><br>
<br><br>
<br><br>
<br><br>

## Start

- If you did not already have the docker group then create it and then **log out and log in again**
```shell
sudo usermod -aG docker $USER
newgrp docker
```


```shell
minikube start \
   --cpus=8 \
   --memory=18G \
   --driver=docker \
   --insecure-registry='xxxxxxxxxxxx' \
   --addons dashboard \
   --addons metrics-server \
   --addons storage-provisioner

MINIKUBE_IP=$(minikube ip)

echo "Minikube IP: $MINIKUBE_IP"

FROM_IP="${MINIKUBE_IP%.*}.$((${MINIKUBE_IP##*.}+1))"
TO_IP="${MINIKUBE_IP%.*}.$((${MINIKUBE_IP##*.}+10))"

# make sure we are in the correct context
kubectl config use minikube

# We have to congiure the loadbalancer IPs manually because of https://github.com/kubernetes/minikube/issues/8283
# We configure the IPs for metallb loadbalancer in minikubes config.json and then start metallb addon. It will read the config and use the IPs for the metallb secret `config`.
# This way we can avoid interactive command `minikube addons configure metallb`, and the ips are persistent.
cat ~/.minikube/profiles/minikube/config.json | jq ".KubernetesConfig.LoadBalancerStartIP=\"$FROM_IP\"" | jq ".KubernetesConfig.LoadBalancerEndIP=\"$TO_IP\"" > ~/.minikube/profiles/minikube/config.json.tmp && mv ~/.minikube/profiles/minikube/config.json.tmp ~/.minikube/profiles/minikube/config.json 
minikube addons enable metallb

_files=$(find  $_directory  | grep 'values')

# make sure Treafik loadbalancer IP is correct in all configs
for file in $_files
do
    while read -r _line; do
    if [ "$_line" != "" ]; then
      # replace loadbalancer ip's with actual ip's from minikube (traefik should always use the first loadbalancer ip)
      sed -E -i "s/([.]*:\s)'([a-z-]*){0,1}([0-9]{1,3}\.){3}[0-9]{1,3}.nip.io'/\1'\2${FROM_IP}.nip.io'/g" $file
    fi
  done <<< $(cat $file | grep 'loadbalancer-IP')  
done

echo "Minikube Loadbalancer can be accessed with ${FROM_IP}.nip.io"

```
- If you are hanging while the start process at the message container creating then try `minikube logs` to check whats going on. When you have firewall rules then you may have a problem here with docker and your firewall
    - Exiting due to DRV_CP_ENDPOINT: Unable to get control-plane node minikube endpoint: failed to lookup ip for ""
      - If you deny all outgoing traffic via ufw make sure to allow outgoing traffic on your desired network adapter to ipv6 range at port 22 that docker has ssh acces
      ```
      sudo ufw allow out on nordlynx from fe80::/64 to any port 22
      ```

















<br><br>
<br><br>
<br><br>
<br><br>

## Drivers
- https://minikube.sigs.k8s.io/docs/drivers/
```shell
minikube start --driver=docker
```

### Docker (container-based)
- **Schnelle Bereitstellung:** Container starten in Sekunden.
- **Ressourcenschonend:** Weniger Overhead als VMs.
- **Leichtgewichtig:** Geringer Speicher- und CPU-Verbrauch.
- **Portabilität:** Docker-Container sind leicht auf verschiedene Systeme übertragbar.

### KVM2 (VM-based)
- **Isolation:** Bietet stärkere Isolation als Container.
- **Kompatibilität:** Gut geeignet für komplexere Workloads oder wenn Hardware-Virtualisierung benötigt wird.
- **Stabilität:** Reif und gut unterstützt in der Linux-Welt.

### Vergleich Docker vs. KVM2
- **Leistung:** Docker ist leichter und schneller, was es ideal für Entwicklungs- und Testumgebungen macht. KVM2 ist robuster und bietet bessere Isolation.
- **Einfachheit:** Docker erfordert weniger Konfiguration und ist einfacher zu verwalten. KVM2 kann komplexer sein, bietet aber mehr Kontrolle über die VM-Umgebung.
- **Anwendungsfall:** Docker eignet sich besser für Anwendungen, die schnelle Iteration und geringere Ressourcen benötigen. KVM2 ist besser für Anwendungen, die stärkere Isolation und Virtualisierung benötigen.

Für die meisten Entwicklungs- und Testaufgaben wäre Docker die bevorzugte Wahl aufgrund seiner Geschwindigkeit und Einfachheit. KVM2 wäre vorzuziehen, wenn eine stärkere Isolation und Kompatibilität mit verschiedenen Workloads erforderlich ist.








<br><br>
<br><br>
<br><br>
<br><br>

## Addons
```shell
minikube start \
   --addons dashboard \
   --addons metrics-server \
   --addons storage-provisioner
```

```
╰─> minikube addons list
# Minikube Add-ons

| Add-on Name                   | Maintainer                     | Beschreibung |
|-------------------------------|--------------------------------|--------------|
| **ambassador**                | 3rd party (Ambassador)         | API-Gateway und Ingress-Controller für Kubernetes. |
| **auto-pause**                | minikube                       | Pausiert automatisch den Minikube-Cluster, wenn er nicht aktiv verwendet wird, um Ressourcen zu sparen. |
| **cloud-spanner**             | Google                         | Integration mit Google Cloud Spanner für verteilte relationale Datenbanken. |
| **csi-hostpath-driver**       | Kubernetes                     | CSI-Treiber zur Verwendung des Hostpfads für Speicher in Kubernetes. |
| **dashboard**                 | Kubernetes                     | Grafische Benutzeroberfläche zur Verwaltung und Überwachung des Kubernetes-Clusters. |
| **default-storageclass**      | Kubernetes                     | Stellt die Standard-StorageClass für den Cluster bereit. |
| **efk**                       | 3rd party (Elastic)            | Elasticsearch, Fluentd und Kibana für Logging und Monitoring. |
| **freshpod**                  | Google                         | Automatische Neustart von Pods, um sicherzustellen, dass sie auf dem neuesten Stand sind. |
| **gcp-auth**                  | Google                         | Authentifizierungs-Plugin für Google Cloud Platform-Dienste. |
| **gvisor**                    | minikube                       | Laufzeit-Umgebung zur Verbesserung der Sicherheit durch Sandbox-Container. |
| **headlamp**                  | 3rd party (kinvolk.io)         | Moderne Benutzeroberfläche zur Verwaltung von Kubernetes-Clustern. |
| **helm-tiller**               | 3rd party (Helm)               | Server-Komponente von Helm zur Verwaltung von Kubernetes-Paketen. |
| **inaccel**                   | 3rd party (InAccel)            | Unterstützung für FPGA-Beschleunigung in Kubernetes. |
| **ingress**                   | Kubernetes                     | Ermöglicht den Zugriff auf Anwendungen von außerhalb des Kubernetes-Clusters. |
| **ingress-dns**               | minikube                       | Stellt DNS-Unterstützung für Ingress-Ressourcen bereit. |
| **inspektor-gadget**          | 3rd party (inspektor-gadget.io)| Werkzeug zur Überwachung und Fehlersuche in Kubernetes-Clustern. |
| **istio**                     | 3rd party (Istio)              | Service-Mesh für Traffic-Management, Sicherheit und Überwachung. |
| **istio-provisioner**         | 3rd party (Istio)              | Unterstützt die Bereitstellung von Istio in Kubernetes. |
| **kong**                      | 3rd party (Kong HQ)            | API-Gateway und Ingress-Controller für Kubernetes. |
| **kubeflow**                  | 3rd party                      | Plattform zur Vereinfachung des Maschinenlernens auf Kubernetes. |
| **kubevirt**                  | 3rd party (KubeVirt)           | Ermöglicht das Ausführen und Verwalten von virtuellen Maschinen auf Kubernetes. |
| **logviewer**                 | 3rd party (unknown)            | Ermöglicht das Ansehen und Durchsuchen von Pod-Logs über eine Weboberfläche. |
| **metallb**                   | 3rd party (MetalLB)            | Load-Balancer für Kubernetes-Cluster, die keine Cloud-Provider-Integration haben. |
| **metrics-server**            | Kubernetes                     | Aggregiert und bietet Zugriff auf Ressourcenmetriken von Nodes und Pods. |
| **nvidia-device-plugin**      | 3rd party (NVIDIA)             | Unterstützung für NVIDIA GPUs in Kubernetes. |
| **nvidia-driver-installer**   | 3rd party (Nvidia)             | Installiert NVIDIA-Treiber auf Nodes. |
| **nvidia-gpu-device-plugin**  | 3rd party (Nvidia)             | Erweiterte Unterstützung für NVIDIA GPUs in Kubernetes. |
| **olm**                       | 3rd party (Operator Framework) | Operator Lifecycle Manager zur Verwaltung von Kubernetes-Operatoren. |
| **pod-security-policy**       | 3rd party (unknown)            | Erzwingt Sicherheitsrichtlinien für Pods im Cluster. |
| **portainer**                 | 3rd party (Portainer.io)       | Benutzerfreundliche Management-Oberfläche für Docker und Kubernetes. |
| **registry**                  | minikube                       | Lokales Container-Image-Registry. |
| **registry-aliases**          | 3rd party (unknown)            | Verwaltet Aliase für Container-Registries. |
| **registry-creds**            | 3rd party (UPMC Enterprises)   | Verwalten von Anmeldeinformationen für Container-Registries. |
| **storage-provisioner**       | minikube                       | Automatisiert die Bereitstellung von Persistent Volumes. |
| **storage-provisioner-gluster**| 3rd party (Gluster)            | Unterstützung für GlusterFS als Speicher-Backend. |
| **storage-provisioner-rancher**| 3rd party (Rancher)            | Unterstützung für Rancher-Storage-Backend. |
| **volumesnapshots**           | Kubernetes                     | Unterstützung für Volume-Snapshots in Kubernetes. |
| **yakd**                      | 3rd party (marcnuri.com)       | Tool zur Bereitstellung von Kubernetes-Anwendungen. |


```


### Dashboard
- **Funktion:** Bietet eine grafische Benutzeroberfläche für die Verwaltung und Überwachung des Kubernetes-Clusters.
- **Nutzen:** Erleichtert die Visualisierung von Ressourcen, das Erstellen und Verwalten von Deployments, Services und anderen Kubernetes-Objekten.
- **Zugriff:** Nach der Aktivierung kann das Dashboard über einen Webbrowser aufgerufen werden.

### Metrics-server
- **Funktion:** Aggregiert und bietet Zugriff auf Ressourcenmetriken (CPU, Speicher) von Kubernetes Nodes und Pods.
- **Nutzen:** Ermöglicht die Überwachung und Skalierung von Anwendungen basierend auf Ressourcenverbrauchsdaten.
- **Verwendung:** Notwendig für die horizontale Pod-Autoskalierung (HPA) und zur Anzeige von Metriken im Kubernetes Dashboard.

### Storage-provisioner
- **Funktion:** Automatisiert die Bereitstellung von Persistent Volumes (PV) in Kubernetes.
- **Nutzen:** Erleichtert das dynamische Erstellen und Verwalten von Speicherressourcen für Pods, ohne manuelle PV-Konfiguration.
- **Verwendung:** Praktisch für Anwendungen, die persistenten Speicher benötigen, wie Datenbanken oder Stateful Applications.
