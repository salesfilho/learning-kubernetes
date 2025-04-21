## Persistent Volume

This step-by-step tutorial teaches you how to create an NFS Persistent Volume in Kubernetes. Refer to the official documentation 
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

### 3. As a devops team member, create a PVC configuration to your applicartion [as seen here.](https://github.com/salesfilho/learning-kubernetes/blob/main/storage/nfs/README.MD).




