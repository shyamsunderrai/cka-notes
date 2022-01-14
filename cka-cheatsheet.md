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

