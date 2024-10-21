```
git clone https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner.git
```

```
kubectl create -f nfs-subdir-external-provisioner/deploy/object/.
```

```
vi pv.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-test
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 100Mi
  claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: data-test
    namespace: default
  nfs:
    path: /data/nfsshare/
    server: 172.168.0.100
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: nfs-client
  volumeMode: Filesystem
```

```
vi pod.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
  spec:
    containers:
    - image: nginx
      name: nginx
      resources: {}
      volumeMounts:
      - mountPath: /var/www/html/
        name: data-test
    dnsPolicy: ClusterFirst
    volumes:
    - name: data-test
      persistentVolumeClaim:
        claimName: data-test
    restartPolicy: Always
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-test
  annotations:
    nfs.io/storage-path: /data/nfsshare
spec:
  storageClassName: nfs-client
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Mi
```
