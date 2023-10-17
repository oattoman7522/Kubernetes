## Join worker node

```
kubeadm token create --print-join-command
```

## Join Master node

```
kubectl get cm kubeadm-config -n kube-system -o yaml
```
```
apiServer:
  extraArgs:
    authorization-mode: Node,RBAC
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: k8s-endpoint:6443
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: v1.23.3
networking:
  dnsDomain: cluster.local
  podSubnet: 10.2.0.0/16
  serviceSubnet: 10.1.0.0/16
scheduler: {}
```
```
kubeadm init phase upload-certs --upload-certs --config kubeadm-config.yaml
```
```
kubeadm token create --print-join-command
```
```
kubeadm join k8s-endpoint:6443 --token 4iegnp.x2tfqdd9gl93zz1o --discovery-token-ca-cert-hash sha256:636786a58df07625167bd7305d56c4cc75bcbedcc474313934aa08c09dc603af --control-plane --certificate-key 6a2f496e172b16584f3700da0427d6e87b3ff06a67383c1bb05cf504128e4465
```

