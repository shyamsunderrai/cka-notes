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