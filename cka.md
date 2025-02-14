
#Docker vs conatainerD

  within k8s we have Container Runtime Interface (CRI) 
  within CRI we have vendor (fournissseur) who is contain Open Container Initiative(OCI)
  and OCI have : imagespec and runtimespec
  
  ctr :    
  nerdctl:
  crictl:
  
# ETC
  run etcd service : ./etcd
  port by default: 2379
  command line client : ./etcdctl (e.g: ./etcdctl set key1 value1)
  
  To set the right version of API set the environment variable ETCDCTL_API command
	export ETCDCTL_API=3
	
# kube-api server
  
  - AUthenticate user
  - Vaidate Request
  - Store and Retrieve data in the ETCD database
  - Update ETCD 
  - Scheduler 
  - Kubelet

# Controler Manager
The controler its the process that 
  - watch the status
  - Remadiate situation
  - 
  
# Pods

# ReplicattionController

# Replicaset 

```sh	
	kubectl create -f replicaset.yml
	kubectl delete -f replicaset.yml
	kubectl get replicaset # kubectl get rs
	kubectl apply -f replicaset.yml
	kubectl replace -f replicaset.yml
	kubectl scale --replicas=8 -f replicaset.yml
	kubectl scale --replicas=8 replicaset replicasetName new-replica-set 
	kubectl scale --replicas=3 replicaset new-replica-set 
```
# Deployment
kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3 -o yaml > deployment-definition.yml 	


# services
kubectl run httpd --image=httpd:alpine 
pod/httpd created

# Delarative methode : 
kubectl config get-contexts
kubectl expose pod httpd --type=ClusterIP --port=80 --name=backend
kubectl run httpd2 --image=httpd:alpine --port=80 --expose

# manuel scheduling
## 1 - Usint of nodeName 
is used to assigne directly the pod to a specific node
example: 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-manual-pod
spec:
  nodeName: worker-node-1  exécuter le pod
  containers:
  - name: busybox-container
    image: busybox
    command: ["sh", "-c", "echo Hello Kubernetes! && sleep 3600"]
```
This pod will be directly assigne to a "orker-node-1"

⚠️ Limitation : Kubernetes does not check if the node has enough resources to run the pod. if it is unavailable, the pod will remain in Pending state.

To check if kubernetes cheduler is present :
```sh
kubectl get pods --namespace kube-system
```
🔹Output: we will show the pod of the scheduller in the list

## 2 - Using nodeSelector

🔹 nodeSelector : allows to assign a pod to a node that has a specific label.
📌 nodeSelector uses node labels to constrain pod scheduling.
✅ Example : 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-manual-pod
spec:
  nodeSelector:
    disktype: ssd  # Only nodes with "disktype=ssd" will execute this pod /its label of node
  containers:
  - name: busybox-container
    image: busybox
    command: ["sh", "-c", "echo Hello Kubernetes! && sleep 3600"]

```
🔹 for apply:
```sh
kubectl apply -f pod-manual.yaml
```
➡️ Kubernetes will only place this pod on a node with the label disktype=ssd

🔹 To see the labels of a node:
```sh	
kubectl get nodes --show-labels
```

# Labels and selector

1 - Labels 
🔹 Labels and Selectors are used to organize, identify, and select Kubernetes objects.

✅ Example of a pod with a label
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
  labels:
    app: my-app
    env: production
spec:
  containers:
  - name: my-container
    image: nginx
```
🔹 Check the labels of a pod:
```sh
kubectl get pods --show-labels
```
🔹 Add a label to an existing pod:
```sh
kubectl label pod nginx-pod version=1
```
🔹 Delete an label
```sh	
kubectl label pod nginx-pod version- 
```
NB : we can add labels in the node using cammand kubectl label

📌 Shows all nodes with their labels.
```sh	
kubectl describe node node-1 | grep "Labels"
```
2 - Selectors : Filter objects using labels
📌 Example of a Service that selects a Pod with a label app=my-app
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app 
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```
🔹 Select pods with a specific label in command line:
```sh
kubectl get pods -l app=my-app
```
🔹 Select multiple labels at once:
```sh
kubectl get pods -l app=my-app,env=production
```
✅ Command to list all Kubernetes resources (Pods, Services, Deployments, etc.) that have the label env=prod.
```sh
kubectl get all --selector env=prod
```
```sh
kubectl get all --selector "env=prod,bu=finance,tier=frontend"
```

##  taint and tolaration

 - If a node has no taints, any pod can be scheduled there.
 - If a node has taints, only pods with matching tolerations can be scheduled on it.
## taint 
```sh
    # taint a node
    kubectl taint nodes nodeName key=value:    
    
    # to display the taints apply to the nodes
    Kubectl describe nodes | grep "taints"
    # to display 5 lignes after each taint
    kubectl describe nodes | grep -A 5 "taints
    # to display 5 lignes before each taint
    kubectl describe nodes | grep -B 5 "taints
    # to display 5 lignes before and after each taint
    kubect describe nodes | grep -C 5 "taints
```	
## tolerations of the pods
for example : 
if the node-1 has the taint :
   ```sh
   kubectl describe ndoes | grep "taints"
   ```	
output : Taints: special=true:NoSchedule

when you describe deployment or pod, if it doesn't have the:
   ```yaml
       tolerations:
       - key: "special"
         operator: "Equal"
         value: "true"
         effect: "NoSchedule"
   ```
✅ Pods CANNOT be scheduled on node-2

```sh
kubectl taint nodes nodeName key=value:taintEffect // taintEffect could be : 
NoSchedule|PreferNoSchedule|NoExecute
NoSchedule|PreferNoSchedule|NoExecute
kubectl describe node kubemaster 
kubectl describe node kubemaster | grep taint // attention
kubectl taint nodes node01 spray=mortein:NoSchedule
untaint the node :
kubectl taint nodes <nom-du-nœud> <clé>-
kubectl taint nodes node1 app-
kubectl taint nodes controlplane node-role.kubernetes.io/control-plane:NoSchedule
```

# Node affinity

Node Affinity is a mechanism to constrain a Pod to be scheduled on a specific node or group of nodes based on their labels. It is a more advanced extension of Node Selectors.

# Type of Node Affinity
There are three types of Node Affinity rules:

🔹 requiredDuringSchedulingIgnoredDuringExecution

- quired: If no node matches the criteria, the Pod will not be scheduled.
- IgnoredDuringExecution means that if the node changes characteristics after scheduling, the Pod is not rescheduled.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
  containers:
  - name: mycontainer
    image: nginx
```
✅ Explanation: This Pod can only be scheduled on nodes with the disktype=ssd label.

🔹 preferredDuringSchedulingIgnoredDuringExecution

-Optional: Kubernetes will favor nodes that meet this constraint but will not block the Pod deployment if no node matches.
- Prioritizes certain nodes based on a weight.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: region
            operator: In
            values:
            - europe-west1
  containers:
  - name: mycontainer
    image: nginx
```
✅ Explication : Kubernetes va privilégier les nœuds situés dans la région europe-west1, mais si aucun n’est disponible, il programmera quand même le Pod ailleurs.

# Static Pods:
to configure the path for the static pod, we need to configure this path in the file :
/var/lib/kubelet/config.yaml and within this file the cofiguration of the path is in the line below : staticPodPath 

# Multiples schedulers

In a cluster kubernetes, it is possible to run multiple instances of the kube-scheduler (the default scheduler) or custom schedulers in parallel. 
it can be for:
- *high availability*: If one instance of the scheduler fails, another can take over. 
- Load distribution : Distribute the planning load accros multiple instances.
- Tests and experimentations : Tests new configuration or version of the scheduler without impacting  production. 
- Personnalized scheduler : Ru multiple scheduer with different scheduling logic.

How setup multiple kube-scheduler instances: 

1 - On a self-hosted kubernetes cluster (On-premise or bare metal)

I you manually manage system services (systemd) on master nodes, you can configure multiple kube-scheduler services.

Step 1: Create additional service 

-  default, the scheduler is managed by a systemd service called kube-scheler.service.
- To ass additinal instances, create service files like kube-scheduler-2.service, kube-scheduler-3.service, etc.
Example service file: /etc/systemd/system/kube-scheduler-2.service
```sh 
[Unit]
Description=Kubernetes Scheduler 2
Documentation=https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-scheduler \
  --config=/etc/kubernetes/scheduler-2-config.yaml \
  --leader-elect=true \
  --leader-elect-resource-name=kube-scheduler-2
Restart=on-failure
User=root

[Install]
WantedBy=multi-user.target
```

Step 2:  Configure each instance

- Each instance should have its own configuration (YAML file) to avoid conflicts.
- Use the --leader-elect=true option to anable leader election and avoid multiple instances scheduling the same pods.
- Use --leader-elect-resource-name to geive each instance a unique name

Example of configuration file: /etc/kubernetes/kube-scheduler-2-config.yaml
```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: kube-scheduler-2
leaderElection:
  leaderElect: true
  resourceName: kube-scheduler-2
clientConnection:
  kubeconfig: /etc/kubernetes/scheduler-2.kubeconfig
```
Step 3: Start Services
```bash
systemctl daemon-reload
systemctl start kube-scheduler-2.service
systemctl enable kube-scheduler-2.service
```
2 - Deploy additional scheduler as a pod

Deploy an additional Kubernetes scheduler as a Pod, along with its associated configuration.

This YAML file describes a pod that hosts a custom scheduler.
my-custom-scheduler.yaml
```yaml 
apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler   # Nom du pod
  namespace: kube-system      # Namespace où le pod sera déployé
spec:
  containers:
  - command:
    - kube-scheduler         # Commande pour démarrer le scheduler
    - --address=127.0.0.1    # Adresse où le scheduler écoute les requêtes
    - --kubeconfig=/etc/kubernetes/scheduler.conf
                              # Fichier kubeconfig utilisé pour se connecter au cluster
    - --config=/etc/kubernetes/my-scheduler-config.yaml
                              # Fichier de configuration spécifique au scheduler
    image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
                              # Image utilisée pour le container du scheduler
    name: kube-scheduler      # Nom du conteneur
```

The next file describes a pod that hosts a custom scheduler.
my-scheduler-config.yaml
```yaml 
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler     # Nom du scheduler (doit être unique)
leaderElection:
  leaderElect: true                 # Active l'élection du leader pour éviter les conflits
  leaderNamespace: kube-system      # Namespace où le leader sera défini
  resourceName: lock-object-my-scheduler
                                    # Nom de l'objet utilisé pour verrouiller l'élection
```
explanation: 
- Scheduler deployment : 
The my-custom-scheduler.yaml file deploys a Pod with a custom instance of the scheduler.
- Personalization : 
The my-scheduler-config.yaml file configures the behavior and settings of this scheduler.

```bash
kubectl create -f my-scheduler.yaml
kubectl get pods --namespace=kube-system
kubectl get events -o wide 
kubectl get sa my-scheduler
kubectl create configmap my-scheduler-config --from-file=/root/my-scheduler-config.yaml -n kube-system
kubectl logs my-custom-scheduler --namespace=kube-system
```

# Admission Controllers
Les Admission Controllers sont des plugins qui interceptent les requêtes à l'API Server 
avant que les objets Kubernetes ne soient persistés dans etcd

📌 Cycle de vie d'une requête dans Kubernetes
- L'utilisateur ou un composant envoie une requête (ex: création d'un pod via kubectl apply).
- L'API Server authentifie et autorise la requête (via RBAC ou d'autres mécanismes).
- Les Admission Controllers interceptent la requête avant qu'elle ne soit stockée.
- Si la requête est validée, l'objet est stocké dans etcd et pris en charge par les autres 
composants (Scheduler, Controller Manager, etc.).

kubectl exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep 'anable-admission-plugins'

kube-apiserver --enable-admission-plugins=MutatingAdmissionWebhook,ValidatingAdmissionWebhook
kube-apiserver --enable-admission-plugins=NamespaceAutoProvision
kubectl exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver --enable-admission-plugins=NamespaceAutoProvision

sudo nano /etc/kubernetes/manifests/kube-apiserver.yaml 

check the process to see enabled and disabled plugins:

ps -ef | grep kube-apiserver | grep admission-plugins

# Mutating and validating admission controllers

Create TLS secret webhook-server-tls for secure webhook communication in webhook-demo namespace.
Certificate : /root/keys/webhook-server-tls.crt
Key : /root/keys/webhook-server-tls.key

```bash 
kubectl -n webhook-demo create secret tls webhook-server-tls \
    --cert "/root/keys/webhook-server-tls.crt" \
    --key "/root/keys/webhook-server-tls.key"
```	

# Rolling update nd rollbacks

Rollout command 
```bash
kubectl create -f deployment-definition.yml # Create 
kubectl get deployments # get 
kubectl apply -f deployment-definition.yml # Update deployment
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1 # update deployment
kubectl rollout status deployment/myapp-deployment # status
kubectl rollout history deployment/myapp-deployment # status
kubectl rollout undo deployment/myapp-deployment # rollback
kubectl get replicasets 
```
# CofigMap
- Les ConfigMaps sont un type de ressource dans Kubernetes qui vous permet de stocker des 
données de configuration sous forme de paires clé-valeur
- Elles sont utilisées pour découpler les configurations des applications des images de 
conteneurs, ce qui permet de gérer et de déployer des applications de manière plus flexible.

## Utilisation des ConfigMaps
- Les ConfigMaps sont utilisés pour stocker des données de configuration non sensibles, 
comme les fichiers de configuration, les variables d'environnement, les commandes de démarrage, etc.

## Création d'un ConfigMap :
- Nous pouvons créer un ConfigMap à partir d'un fichier, d'un répertoire ou de littéraux directement 
dans un fichier YAML.
Exemple: 
```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-config
data:
  config.json: |
    {
      "key": "value"
    }
  ENV_VAR: "value"
```

kubect create cm my-config-map --from-literal=key=value --fron-literal=kay=value

# Chiffrement des secrets (ou tout autre objets k8s)
Si vous exécutez kube-apiserver avec l'argument --encryption-provider-config de ligne de commande

1- create the secret ///////////////////////////////
2- installe le client etcd si nécessaire: sudo apt install etcd-client 
à l'aide de ll'outil etcdctl, lire ce secret dans etcd

ETCDCTL_API=3 etcdctl get /registry/secrets/default/secret1 [...] | hexdump -C
ou 
ETCDCTL_API=3 etcdctl \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key  \
   get /registry/secrets/default/secret1 | hexdump -C


3- create the encryption configuration yaml file with the right provider 
    Exemple
```yaml
---
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
      - configmaps
      - pandas.awesome.bears.example
    providers:
      - aescbc:
          keys:
            - name: key1
              # See the following text for more details about the secret value
              secret: <BASE 64 ENCODED SECRET>
      - identity: {} # this fallback allows reading unencrypted secrets;
                     # for example, during initial migration
```

kubectl -n elastic-stack logs kibana


kubectl drain node01 --ignore-daemonsets
k uncordon nodeO1
k cordon node

# Upgrade proce

to upgrade the cluster 
```bash	
kubeadm upgrade plan # pour voir le plan des mise à jour  
apt-get upgrade -y kubeadm=1.12.0-00 # pour mettre à jour le kubeadm, ce ci doit etre fait avant la mise à jour du cluster
kubeadme upgrade apply # ceci met à jour le cluster  (la mise à jour du control plan), cela met àjour le kube-apiserver
# ou excéuter la commande 
kubeadme upgrade apply 1.12.0

```	
The next step is to uprade the kubelet
```bash
kubectl get nodes # nous montre la verrsion des noeuds (des kubelet)
# En fonction de notre installation, nous pouvons avoir le kubelet dans notre master node 
# car on peut avoir le kubeadm déployé comme kubelet sur le master node pour 
# exécuter les composants du controlplan comme pods dans le master node

# si on a le kubelet dans le master node, pour le mettre à jour, exécuter :
apt-get upgarde -y kubelet=1.12.0-00 
# une fois mise à jour restart le service kubelet
systemctl restart kubelet
#Pour verifier si cela à bien été mit à jour
kubectl gt nodes

# Dans la suite nous devons upagrade les kubelet dans les workers nodes
# Dans cette partie, nous devons avoir une stratégie de mise à jour si nous avons des applications 
# qui tournes dans ces nodes 
# Par exemple mettre à jour un node à la fois
# mise à jour du node-1 par exemple :
kubectl drain node-1 # pour déplacer les pods vers d'autres nodes, et le rend unschedulable
apt-get upgrade -y kubeadm=1.12.0-00 # met à jour le kubeadm du node-1
apt-get upgrade -y kubelet=1.12.0-00 # met à jour le kubelet du node-1
kubeadm upgrade node config --kubelet-version v1.12.0 
systemctl restart kubelet
# pour voir la version : 
kubectl get nodes
# Dans le suite nous devons rendre le node schedulable
kubectl uncordon node-1

# pour chaque node, effetcuer les meme oprations
