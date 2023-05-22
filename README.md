# Kubernetes Commands Cheatsheet And Notes
# Undefined
* Declarative = setup using text editor
* Imperative = setup using commandline

* Symmetric encryption = single key
* Asymmetric encryption = private and public key

Best practices:
- Use custom user id 1000 > incremental


> Ctrl + r = search history
```
apt list --installed | grep kubeadm     # List installed packages named kubeadm

apt-get madison PACKAGE                 # List available packages in the repo
```
# Vim
```
~/.vimrc
set tabstop=2
set expandtab
set shiftwidth=2
syntax on
```

OR

```
cat <<EOF | tee ~/.vimrc
set tabstop=2
set expandtab
set shiftwidth=2
syntax on
EOF
```

# Aliases
```
alias k='kubectl'
alias kn='kubectl config set-context $(kubectl config current-context) --namespace'
alias kc='kubectl config get-contexts'
alias kw='watch -n 2 "kubectl get all,cert,svc,ing,cm,secret,sa"'
```

# Show current context
```
kubectl config view
```

# History
## Show previous used commands
```
history
```

## Use number 7 från history output
```
!7
```

# Delete multiple resources
```
kubectl delete pod nginx nginx2
```

# Various commands for systemctl
```
systemctl status kubelet
systemctl start kubelet
systemctl stop kubelet
systemctl cat SERVICE/UNIT FILE
```
# Check logs for service via journalctl
```
journalctl -u kubelet
```
# List API resources
```
kubectl api-resources
kubectl api-resources --namespaced
```
# Access resources using client certifcate, private key and
```curl -k OR wget --no-check-certificate can be used to override untrusted CA.```
```
curl https://my-k8s-cluster:6443/api0/v1/pods --cert client.crt --key private.key --cacert ca.cert

wget https://my-k8s-cluster:6443/api0/v1/pods --certificate=client.crt --private-key=private.key --ca-certificate=ca.cert
```
# View all resources in a namespace
```
kubectl get namespace,node,ingress,pod,svc,job,cronjob,deployment,rs,pv,pvc,secret,ep -o wide
```
# Readyness and liveness probes
Check state through command execution, HTTP GET or TCP.

* Readyness = Restarts the POD if state check is failed.
* Liveness = Doesn't serve traffic to the POD if state check is failed

# Export YAML
```
kubectl --dry-run=client -o yaml
```

# Work with contexts
```
kubectl config set-context $(kubectl config current-context) --namespace NAMESPACE
kubectl config current-context
kubectl config get-contexts -o name > /opt/course/1/contexts
```

# Make a variable for exporting to YAML
```
export output="--dry-run=client -o yaml"
```

# Interactive shell into POD
```
kubectl exec -it POD -- sh                      
```

# Force deletion
```
export now="--force --grace-period 0"

kubectl delete pod POD $force
```

# Sort output by metadata
```
kubectl get pod -A --sort-by=.metadata.creationTimestamp
```

# Test user rights
```
kubectl auth can-i -as=USER
kubectl auth can-i -as=system:serviceaccount:kube-system:default
```

# Record command
```
kubectl get pods -A --record
```

# Show prec
```
kubectl rollout history
kubectl rollout --to-revision=X
```

# Create a pod with a command and args
```
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
  labels:
    purpose: demonstrate-command
spec:
  containers:
  - name: command-demo-container
    image: debian
    command: ["/bin/sh","-c"]
    args: ["command one; command two && command three"]
  restartPolicy: OnFailure
```

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  securityContext:
    runAsUser: 1010
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
```
# Service vs Ingress vs Endpoints

Service = logical construction of one of more pods
Ingress = level 7 load balancer
Endpoints = IP + port tied to a resource

# Service
Logical representation of a app between nodes

```! Default TCP```
```Headless service = service utan IP som pekar direkt på poddar med DNS```

* NodePort = Export service to a specified port on each node (default 30000-32767)
* ClusterIP = Exists only i cluster
* loadbalancer = HA proxy eller liknande + loadbalancer ELLER Azure, GCP, AWS + loadbalancer

# Create a resource and expose it to specified port
```Port = Port open for the container. This port is exposed to the service.```

```TargetPort = The actual port which you application inside the container is listening on```
```
kubectl run httpd --image=httpd:alpine --port=80 --expose                 # Creates pod and service listening on port 80
kubectl run custom-nginx --image=nginx --port=80                          # Creates pod that are listening on port 80

kubectl expose pod redis --port=6379 --name redis-service                 # Exposes pod at port 6379 and create a service with the name redis-service
kubectl expose deployment nginx --port=80 --target-port=8080              # Exposes deployment at port 80 with a container port 8080. 

kubectl expose deployment hr-web-app --port=80 --type=NodePort          # Exposes deployment at containerport 8080 and randomize an port for NodePort
```

# Create secret imperative 
```
kubectl create secret generic prod-db-secret --from-literal=username=USER
--from-literal=password=1234

kubectl create secret generic db-user-pass2 \
  --from-file=username=./username.txt \
  --from-file=password=./password.txt
```
# Paths for Kubernetes config and logs
! kube-dns prior to 1.12 and core-dns from 1.12
```
/etc/kubernetes/
    /etc/kubernetes/kubelet.conf
    /etc/kubernetes/controller-manager.conf
    /etc/kubernetes/scheduler.conf

/etc/kubernetes/manifests/
    /etc/kubernetes/manifests/kube-apiserver.yaml
    /etc/kubernetes/manifests/kube-controller-manager.yaml
    /etc/kubernetes/manifests/kube-scheduler.yaml
    /etc/kubernetes/manifests/etcd.yaml

/etc/kubernetes/pki/

/etc/cni/net.d/
```


## Control plane
```
/var/log
    /var/log/kube-apiserver.log
    /var/log/kube-scheduler.log
    /var/log/kube-scheduler.log/var/log/kube-controller-manager.log
```

## Worker
```
/var/log
    /var/log/kubelet.log
    /var/log/kube-proxy.log
```
## Display certificate
```
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

# CoreDNS
! kubelet sets CoreDNS as default and add search suffix such as: default.svc.cluster.local svc.cluster.local cluster.local
/etc/coredns/CoreFile
kubectl describe configmap coredns -n kube-system

# When API server is unavialble
```
critctl ps -a
```

# Output info. from jsonpath
```
kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/FILE.txt
kubectl config view -o jsonpath="{.contexts[*].name}" | tr " " "\n"

kubectl get nodes -o jsonpath={'.items[*].status.addresses[0].address'}

kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'

kubectl get nodes -o jsonpath='{.items[*].status.addresses[0].address}'

kubectl get nodes -o json | jq .items[].status.addresses[0].address

kubectl get deploy -o=custom-columns=NAME:metadata.name,CONTAINER_IMAGE:spec.image 
```

# Run kubectl with different kubeconfig
```! A users kubeconfig is by default stored under $HOME/.kube/config```

```! Upon creation of the cluster a special kubeconfig is created at /etc/kubernetes/admin.conf, which can be used to achieve ANYTHING with the cluster.```
```
kubectl --kubeconfig FILE
```
# performance node req. metric server
```
kubectl top node
kubectl top pod
kubectl top pod --containers
```

# Get logs about nodes, pods, deployments, replicasets etc.
```
kubectl logs
```
# stream logs of pod/containers
```
kubectl logs -f NAMEOFPOD 
kubectl logs -f NAMEOFPOD NAMEOFCONTAINER
```
# Get cluster events of current or all namespaces
```
kubectl get events
kubectl get events -A
```
# Create role, clusterrole and rolebinding 
* Role = specific to a namespace
* Cluster role = applies to whole cluster

```
kubectl create role developer --resource=pods --verb=create,list,get,update,delete
kubectl create rolebinding developer-role-binding --role=developer --user=john
```
# RBAC
Authenication
* Static Password File
* Static Token File 
* Certificates
* Identify Services

Authorization
* RBAC
* ABAC
* Node
* Webhook

```! Is processed in order.```

```--authorization-mode=Node,RBAC,Webhook```

# DNS namespace
```
db-service.dev.svc.kubecluster.local
RESOURCE.NAMESPACE.RESOURCETYPE.CLUSTERNAME.DOMAIN
```


# Create config map
```
kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue
```


# Encode or decode base64
```
echo -n 'admin' | base64
echo -n '' | base64 --decode
base -w 0 FILE > OUTPUT # output file with only on line
```

# Secrets
```
kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123

kubectl describe secret NAMEOFACCOUNT
```
token = key



```
kubectl create serviceaccount pvviewer
kubectl create clusterrole pvviewer-role --resource=persistentvolumes --verb=list
kubectl create clusterrolebinding pvviewer-role-binding --clusterrole=pvviewer-role --serviceaccount=default:pvviewer
```
```! Each namespace has it's own default service account```

```

kubectl create serviceaccount NAMEOFACCOUNT
kubectl get serviceaccount
```


# Deployment options

* Statefulset = typ Databas eller daemons
* Replicaset = HA för containers
* Deployments = HA för container + extra features. <- most common


# Deployments 
## The basics
```
kubectl create deployment hr-web-app --image=nginx --replicas=2
```

## Rolling updates
```--record = is good practice to used when updating deployments```
```
kubectl rollout status deployment = status för deployment

kubectl rollout history deployment = history för deployment

kubectl rollout undo deployment (--to-revision=2) = rullar tillbaka en deployment

kubectl rollout pause deployment

kubectl rollout resume deployment
```

* Deployment ReCreate = Avbryter pods och ersätter med ny version.
* Deployment  Rolling Update = Spinner upp nya poddar vid sidan om och ersätter stegvis äldre version.

# Examples of ingress controllers
- Nginx
- Traefik
- HAProxy

# misc
```
kubectl explain = förklarar olika möjliga parametrar
kubectl exec -it = "loggar in på pod/container"
kubectl get pod --all-namespaces/-A
kubectl get pod -n NAMEOFNAMESPACE
kubectl get all --selector env=prod,bu=finance
```
Pod:
- Single container
- Multiple containers
    - Sidecar
    - Adapter
    - Ambassador

# Pod status

* Pending = scheduler funderar vart poden ska placeras.
* ContainerCreating = pod skapas
* Running = pod som körs
* Terminating = pod är avstängd.

* PodScheduled = när poden är schedulerad
* Initialzed = när det är påbörjad.
* ContainersReady = när alla containrar i en pod är redo
* Ready = alla containrar i en pod

# Labels, selectors and annotations
* Labels = "tagg"
* Selectors = "filter"
* Annotations = Systemspecifik metadata t.ex. ```kubernetes.io/hostname=node01```

```
kubectl run NAMEOFPOD --image=NAMEOFIMAGE -n NAMESPACE --labels=tier=db             # Start POD with the label tier with value db

kubectl label pods my-pod new-label=awesome                      # Add a Label
kubectl annotate pods my-pod icon-url=http://goo.gl/XXBTWq       # Add an annotation
```
```
metadata:
  name: simple-app
  labels:
    test: 1
    function: web
```

# Label nodes 
```
kubectl label nodes node01 LABEL=VALUE
```

# Network Policies
```! By default - all allow```

# Architecture overview
> Odd number of nodes is prefered. 3-5.
## Masters
- API Server = svarar på API
- Scheduler  = Väljer optimal nod för placering av last beroende på definition. Vad lasten ska startas, men startar inte lasten.
- Controller Manager = onboardar nya noder, kontrollerar rätt antal pods körs. 
- etcd (2370 TCP) = state of the cluster

## Worker/Nodes
- Kubelet = agent som pratar med API Server
- Kube-Proxy = nätverk modul som hanterar nätverk som säkerställer regelverk
- Container Runtime (docker) 



# Volumes, PV, PVC etc.
> readOnly = true, can be use to only provide read only access to a container even though pvc/pv provides write access.
> allowVolumeExpansion can set on certain storageclasses 

volume mount = skapas automatiskt av Docker
bind mount = sökväg till plats


volume = for each pod
persistent volume = manage storage centrally
persistent volume claim = claim volume from persistent volume

```
kubectl get persistentvolume OR kubectl get pv
kubectl get persistentvolumeclaim OR kubectl get pvc
```

! if a PVC is deleted, by default it is retained
persistentVolumeReclaimPolicy: retain


! if PVC is deleted, the PV is delete aswell and then available to other PVC
persistentVolumeReclaimPolicy: delete


! if PVC is deleted, the PV will be scrubbed and the available to other PVC
persistentVolumeReclaimPolicy: Recycle

kubectl get storageclasses



Static Provisioning - first create storage e.g. GCP and then a PV
Dynamic Provisioning - create storageclass object and then provide PVC with storageclassname

# Taints and Tolerations + Node affinity
Taint = On node
Tolerations = On PODs

* Taints and Tolerations = Matchar PODs med noder, men garanterar inte att PODarna hamnar på önskade noder.
* nodeSelectors = Matchar PODs med noder, baserat på en specifik label
* Node Affinity = Matchar PODs med noder, men noder kan få annan oönskad last.
* Taints and Tolerations + Node Affinity = matchar pods med noder och underviker oönskad last 

```NoSchedule = Don't schedule PODS, don't kill already running PODs```

```PreferNoSchedule = Prefer not to schedule PODS, don't kill already running PODs```

```NoExecute = Don't schedule PODS, evects = kills PODs```
## Taint and tolerations
```
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "spray"
    operator: "Exists"
    value: "mortein"
    effect: "NoSchedule"
```
```
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
```
```
kubectl taint node NODE color=blue:NoSchedule               # Add taint
kubectl taint node NODE color=blue:NoSchedule-              # Remove taint
```

## Node affinity
```
    spec:
      containers:
      - name: blue
        image: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In
                values:
                  - blue
```
```
    spec:
      containers:
      - name: red
        image: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/master
                operator: Exists
```
```
    spec:
      containers:
      - name: red
        image: nginx
       affinity:                                           
          podAntiAffinity:                                 
            requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:                               
                matchExpressions:                          
                - key: id                                  
                  operator: In                             
                  values:                                  
                  - very-important                         
              topologyKey: kubernetes.io/hostname          

```
# Save etcd snapshot
```
ETCDCTL_API=3 etcdctl --endpoints x.x.x.x:2379 \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \snapshot save ~/snapshot
```

# Restore etcd snapshot
```! According to the official documentation it's recommanded also that the kube-scheduler, kube-controller-manager and kubelet is restarted to ensure that these service doesn't rely on stalled data.```

```
1. service kube-apiserver stop OR move temporarily /etc/kubenetes/kube-apiserver.yaml to different location.
2. ETCDCTL_API=3 etcdctl --endpoints x.x.x.x:2379 \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \snapshot save ~/snapshot
3. service etcd stop OR move temporarily /etc/kubenetes/etcd.yaml to different location.
4. update the hostpath for etcd databas in /etc/kubernetes/manifests/etcd.yaml
5. ETCDCTL_API=3 etcdctl --data-dir /var/lib/etcd-from-backup --endpoints x.x.x.x:2379 \
  --cert=pki/etcd/server.crt \
  --key=pki/etcd/server.key \
  --cacert=pki/etcd/ca.crt \snapshot restore ~/snapshot
6. service etcd start etcd OR move back the manifest file for etcd to it's original location.
7. service kube-apiserver start or move back the manifest file for kube-apiserver to it's original location.
```

# Kubernetes versions
```! 3 minor updates are supported.```

```! When upgrading, consider upgrading one minor version at the time.```
```
v1      .11.    3
MAJOR   MINOR   PATCH
```

Minor 
- Features
- Functionalities

Patch
- Bug fixes

# Show current version
```
kubeadm upgrade plan
```

# Upgrade to next version
```
kubeadm upgrade apply v1.20.03
```
# Print token for joining cluster
```
kubeadm token create --print-join-command
```

# Join node to cluster
kubeadm join 
kubeadm token create --print-join-command
kubeadm join --token --discovery-token-ca-cert-hash --certificate-key --control-plane

# Upgrade cluster
```! The masters first, then worker nodes.```

## Master
```
1. kubectl drain HOSTNAME --ignore-daemonsets

2. apt-get update

3. apt-mark unhold kubeadm kubelet kubectl

4. apt-get install kubeadm=x.xx.xx-xx

5. kubeadm upgrade apply vx.xx.xx

6. apt-get install kubelet=x.xx.xx-xx

7. apt-mark hold kubeadm kubelet kubectl

8. systemctl daemon-reload

9. systemctl restart kubelet

10. kubectl uncordon HOSTNAME

```

## Worker/Node
```
1. kubectl drain HOSTNAME --ignore-daemonsets

2. apt-get update

3. apt-mark unhold kubeadm kubelet kubectl

4. apt-get install kubeadm=x.xx.xx-xx

5. kubeadm upgrade node

6. apt-get install kubelet=x.xx.xx-xx kubectl=x.xx.x-xx

7. apt-mark hold kubeadm kubelet kubectl

8. systemctl daemon-reload

9. systemctl restart kubelet

10. kubectl uncordon HOSTNAME
```

# Drain node
! If you try to drain a node that have an app, not belong to a replicaset you recive an error.
```
kubectl drain HOSTNAME
```
# Make node unscheduable
```
kubectl cordon HOSTNAME
```
# Make node scheduable (PODs arent moved here automatically)
```
kubectl uncordon HOSTNAME
```