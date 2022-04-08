
# ETCD Notes for CKA prep

This is mostly optional, but good to know. 
**etcd** is the name of tool start the key/value database


#### Starting etcd Cluster
```bash
which etcd
root@samurai:~# which etcd
/usr/bin/etcd
root@samurai:~# /usr/bin/etcd
```

#### Checking cluster health using etcdctl
```bash
root@samurai:~# etcdctl cluster-health
member 8e9e05c52164694d is healthy: got healthy result from http://localhost:2379
cluster is healthy
```

#### Get/Set key/value pair options in etcd
```bash
root@samurai:~# etcdctl set k01 val01
val01
root@samurai:~# etcdctl get k01 val01
val01
```

#### Creating a directory in etcd and then adding key/values within
```bash
root@samurai:~# etcdctl mkdir mydir 
root@samurai:~# 
root@samurai:~# etcdctl set /mydir/mykey myvalue
myvalue
root@samurai:~# etcdctl get /mydir/mykey myvalue
myvalue
```

#### Using mk to create a new key with existing value
```bash
root@samurai:~# etcdctl mk /mydir/newkey myvalue
myvalue
root@samurai:~# etcdctl get /mydir/newkey myvalue
myvalue
root@samurai:~# etcdctl get /mydir/mykey myvalue
myvalue
```

#### Functions of rmdir 
```bash
root@samurai:~# etcdctl rmdir /mydir
Error:  108: Directory not empty (/mydir) [20]
root@samurai:~# 
root@samurai:~# etcdctl rm /mydir/mykey
PrevNode.Value: myvalue
root@samurai:~# etcdctl rm /mydir/newkey
PrevNode.Value: myvalue
root@samurai:~# etcdctl get /mydir/mykey
Error:  100: Key not found (/mydir/mykey) [22]
root@samurai:~# etcdctl get /mydir/newkey
Error:  100: Key not found (/mydir/newkey) [22]
root@samurai:~# 
root@samurai:~# etcdctl rmdir /mydir
```


#### Usage of ls/set within etcdctl
```bash
root@samurai:~# etcdctl mkdir /mydir
root@samurai:~# etcdctl ls /mydir
root@samurai:~# etcdctl set /mydir/k1 val001
val001
root@samurai:~# etcdctl set /mydir/k2 val002
val002
root@samurai:~# etcdctl ls /mydir
/mydir/k1
/mydir/k2
```

#### Usage of setdir and its --ttl property
```bash
root@samurai:~# etcdctl setdir /mydir/k1 --ttl 10
root@samurai:~# etcdctl ls /mydir
/mydir/k1
/mydir/k2
root@samurai:~# etcdctl ls /mydir
/mydir/k1
/mydir/k2
root@samurai:~# etcdctl ls /mydir
/mydir/k2
```

#### Use of update with to set a new value for existing key
```bash
root@samurai:~# etcdctl ls /mydir/
/mydir/k2
root@samurai:~# etcdctl get /mydir/k2
val002
root@samurai:~# etcdctl update /mydir/k2 val000002
val000002
root@samurai:~# etcdctl get /mydir/k2
val000002
```

#### Usage of updatedir
```bash
root@samurai:~# etcdctl set /mydir/k1 val001 
val001
root@samurai:~# etcdctl ls /mydir
/mydir/k1
root@samurai:~# etcdctl updatedir --ttl 30 /mydir/k1 val01
root@samurai:~# etcdctl ls /mydir
/mydir/k1
root@samurai:~# etcdctl ls /mydir
```

#### Using watch
> Allows to watch the changes happening on a given key
```bash
root@samurai:~# etcdctl ls /mydir
root@samurai:~# etcdctl set /mydir/k1 val001
val001
root@samurai:~# etcdctl watch /mydir/k1 
valxxxxx
```

> "-f" is similar to tailing a log 
```bash
root@samurai:~# etcdctl watch -f /mydir/k1 
valyyyy
valzzzz
```

#### etcdctl member function -- to expand
```bash
root@samurai:~# etcdctl member --help
NAME:
   etcdctl member - member add, remove and list subcommands

USAGE:
   etcdctl member command [command options] [arguments...]

COMMANDS:
   list    enumerate existing cluster members
   add     add a new member to the etcd cluster
   remove  remove an existing member from the etcd cluster
   update  update an existing member in the etcd cluster

OPTIONS:
   --help, -h  show help
   
root@samurai:~# etcdctl member list
8e9e05c52164694d: name=default peerURLs=http://localhost:2380 clientURLs=http://localhost:2379 isLeader=true
```

#### Backup and Restore of etcd
Taking backup in etcd
```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db
service kube-apiserver stop
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db --data-dir /location/to/etcd/dir
systemctl daemon reload
systemctl restart etcd
systemctl kube-apiserver start
```
> Command to take the etcd backup
```bash
ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379  --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/snapshot-pre-boot.db
```
> To Restore
```bash
ETCDCTL_API=3 etcdctl  --data-dir /var/lib/etcd-from-backup snapshot restore /opt/snapshot-pre-boot.db
```






