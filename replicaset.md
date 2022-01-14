# ReplicaSet (RelicationController) 

Sample available in "2-replicationcontroller.yaml" for ReplicationController (replaced by ReplicaSet)
Sample of replicaset available in "3-replicaset.yaml"

> Some important notes 
<ul>
<li>ReplicaSets must have the selector tab for the replicaset to know which pods to monitor</li>
<li>To scale a replicaset, we can do the following</li>

```bash
kubectl edit rs "replicasetname" & modify the value for replicas from "x" to "n"
kubectl scale --replicas=3 -f replicaset-file.yaml
kubectl scale --replicas=3 replicaset "replicasetname" 
```
</ul>

