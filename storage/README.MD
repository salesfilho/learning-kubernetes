## Persistent Volume

This step-by-step tutorial teaches you how to create a NFS Persistent Volume in Kubernetes. Refer to the official documentation 
at https://kubernetes.io/docs/concepts/storage/persistent-volumes/ for more details.

### 1. As a system administrator, create and export the NFS directory [as seen here.](https://github.com/salesfilho/learning-kubernetes/blob/main/storage/nfs/README.MD)

### 2. As a cluster administrator, create a PV configuration

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0001
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: fast-ssd
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /sorage01
    server: master01
```

Run this command:

```
microk8s.kubectl apply -f nfs-pv.yml
```

The result is something like:

```
$ microk8s.kubectl get pv
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv0001   10Gi       RWX            Retain           Available           fast-ssd       <unset>                          22s
```


### 3. As a devops team member, create a PVC configuration to your applicartion [as seen here.](https://github.com/salesfilho/learning-kubernetes/blob/main/storage/nfs/README.MD).




