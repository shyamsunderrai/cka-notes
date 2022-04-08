# CheatSheet for Kubernetes

To create a pod with image as "nginx:latest" and with label "app=testapp"
```bash
kubectl run mytestpod --image=nginx:latest --labels='app=testapp'
```

To get the pod yaml only without creating
> kubectl run mytestpod --image=nginx:latest -l 'app=mytestapp' --dry-run=client -o yaml

```bash
kubectl run mytestpod --image=nginx:latest -l 'app=mytestapp' --dry-run=client -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    app: mytestapp
  name: mytestpod
spec:
  containers:
  - image: nginx:latest
    name: mytestpod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

To get deployment yaml definition 
> kubectl create deployment mydeploy --image=nginx --dry-run=client -o yaml
> OR the following for more replicas
> kubectl create deployment mydeploy --image=nginx --replicas=3 --dry-run=client -o yaml 

```bash
kubectl create deployment mydeploy --image=nginx --dry-run=client -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: mydeploy
  name: mydeploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mydeploy
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: mydeploy
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

Format to connect to a service on a given namespace
> servicename.namespace.svc.cluster.local
> Here "servicename" is the service created identified by its name & "namespace" is the actual namespace like "default"
> cluster.local is the default domain for kubernetes cluster


To change the default namespace
> kubectl config set-context $(kubectl config current-context) --namespace=dev
> This would ensure that the default namespace is now set to "dev"


Scheduling --- 
> To check a pod or set of resources using labels, we can use selector
```bash
kubectl get pods --selector app=dev 
```
> Selecting pod/resources with multiple labels
```bash
kubectl get pods --selector tier=frontend,env=prod,bu=finance
```

Command to Taint nodes, so deployments of pods become intolerant
```bash
kubectl taint nodes node01 spray=mortein:NoSchedule
```
> To check the taints
```bash
kubectl describe node <nodename> | egrep -B5 Taints
```

Apply node labels, imperative approach
```bash
kubectl label nodes <nodename> key=value
```
> To apply the affinty rules to deployment
> Aligns with the same identation as containers, inside spec
```bash
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In
                values:
                - blue
      containers:
      - image: nginx
```                  

Modifiable fiends of POD are
>spec.containers[*].image
>spec.initContainers[*].image
>spec.activeDeadlineSeconds
>spec.tolerations

To get all the events from the default or a given namespace
> kubectl get events
> kubectl get events -n <namespace>

To sort resources after enabling metrics-server
> kubectl top pod --sort-by='memory'
> kubectl top node --sort-by='memory'

Under application lifecycle management, Rollout is useful
> kubectl rollout status deploy/<deployment-name> // To check if the deployment went successfully
> kubectl rollout history deploy/<deployment-name> // To check the version of deployments 
> kubectl apply -f deploymentfile.yaml // To deploy changes to an existing file
> kubectl set image deployment/<deployment-name> nginx=nginx:1.18.0

Commands & Entrypoint
> Commands passed in docker image can be completely overwritten from the interactive aspect, when it is passed as entrypoint
> Entrypoint, appends the values
> For instance, ENTRYPOINT['sleep'] -- docker run myimage 10 // This would append the value 10 in front of sleep 
> whereas, for CMD['sleep','10'] -- docker run myimage sleep 20 // The whole command can be overwritten
> "command" field in pods overrides ENTRYPOINT
> "args" field in pods override CMD 

ConfigMap
> kubectl create configmap <config-map-name> --from-literal=COLOR=blue // here we can pass environment variable COLOR and its value as blue
> Multiple such entries can be passed
> Better way to create a config map would be using a file
> kubectl create configmap <config-map-name> --from-file=/path/to/file
> Variables can be loaded via
```bash
envFrom:
  - configMapRef:
      name: <config-map-name>
```
OR
```bash
env:
  - name: COLOR
    valueFrom:
       configMapKeyRef:
         name: app-config
         key: COLOR
```
OR
```bash
volumes:
- name: app-config-volume
  configMap:
    name: app-config
```

SecretRef: Mostly same as configMap

> Variables can be loaded via
```bash
envFrom:
  - secretRef:
      name: <secretRef-name>
```
OR
```bash
env:
  - name: COLOR
    valueFrom:
       secretKeyRef:
         name: secret-file
         key: COLOR
```
OR
```bash
volumes:
- name: app-secret-volume
  secret:
    secretName: app-secret
```

OR as volumes
```bash
volumes:
- name: app-secret-volume
  secret: 
    secretName: app-secret 
```    


Cluster Maintenance
> kubectl drain <nodename>
> kubectl cordon <nodename> // Makes node unavailable for new deployments
> kubectl uncordon <nodename>


Cluster version
> kube-api version is superclass
> controller-manager & kube-scheduler can be of version -1 or -2
> kubelet & kube-proxy can be of versions -1 -2 or -3
> kubelet can be +1 or -1
> Only 3 versions are supported at a time like v.10, v.11 and v.12
> Upgrade is sequential to these releases

Upgrading Cluster
> apt-get upgrade -y kubeadm=1.12.0-00
> kubeadm upgrade apply v1.12.0
> To upgrade kubelet
> apt-get upgrade -y kubelet=1.12.0
> systemctl restart kubelet

Steps to upgrade a worker node
> kubectl drain node01
> apt-get upgrade -y kubeadm=1.12.0-00
> apt-get upgrade -y kubelet=1.12.0-00
> kubeadm upgrade node config --kubelet-version v1.12.0
> systemctl restart kubelet
> kubectl uncordon node01


Backup methodologies
> Velero - to backup kubernetes cluster
> 


Security
> Authentication mechanisms 
> Static Password File, Static Token file, Certificates or Identity Services (LDAP)
> Static Password File [password, username, userid, groups]
> kube-apiserver, start with --basic-auth-file=/path/to/static/password/file ### From systemctl or kube-apiserver.yaml
> To test, curl can be used for authentication

```bash
curl -v -k https://master-node-ip:6443/api/v1/pods -u "user:password" 
```

> Static Token File [token, username, userid, groups]
> kube-apiserver, start with --token-auth-file=/path/to/static/token/file ### From systemctl or kube-apiserver.yaml
> To test, curl can be used for authentication

```bash
curl -v -k https://master-node-ip:6443/api/v1/pods --header "Authorization: Bearer <token"
```
**Either are not recommended as they are stored in clear text**

> PKI - Public key interface
> Nomenclature
> server.crt OR server.pem ## Server PUBLIC Certificate
> client.crt OR client.pem ## Client PUBLIC Certificate
> server.key OR server-key.pem ## Server PRIVATE Certificate
> client.key OR client-key.pem ## Client PRIVATE Certificate

Server Components
> kubeapi server
> etcd server
> kubelet server

Client Components
> Administrators (to run kubectl)
> kube-scheduler // for scheduling 
> kube-controller-manager // for controlling resources on k8s
> kube-proxy // to auth with kube-api

**kube-api server is the client for both etcd and kubelet**

Generating certificates for cluster
> Common tools (easyrsa, openssl, etc )
> Creating SSL signed certificate
```bash
openssl genrsa -out ca.key 2048
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr // Certificate signing request
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt // Signing the csr using private key, hence self-signed certificate
```

Generating Client Certificate
```bash
openssl genrsa -out admin.key 2048
openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
```



