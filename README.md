# HomeLab - MicroK8S

## Installation

### Prepare the nodes

#### Install nfs-common
This package is necessary for shared storage over the network, which enables distributed applications to access the same storage from multiple nodes.
```
sudo apt install -y nfs-common
```

#### Install ipvsadm
This tool is required for configuring load balancing using IPVS (IP Virtual Server), which is more efficient for handling traffic across Kubernetes nodes.
```
sudo apt install -y ipvsadm
```
Edit `/etc/modules-load.d/ipvs.conf`
```
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
```

Load IPVS modules without rebooting 
```
sudo systemctl restart systemd-modules-load.service
```

### Install the nodes
Install the nodes
```
sudo snap install microk8s --classic --channel=1.32
```

Create cluster
```
microk8s add-node
```
Issue `microk8s join` commands that you get from previous command on **other** nodes.



### Install kube-vip
Documentation: https://kube-vip.io

Create file `kube-vip.cnf`
```
config:
  address: "192.168.1.40"
env:
  vip_interface: "eno1" # This is the interface, same on all nodes where the VIP is announced
  vip_arp: "true"  # mandatory for L2 mode
  lb_enable: "true"
  lb_port: "16443" # changed as microk8s uses 16443 instead of 6443
  vip_cidr: "24"
  cp_enable: "true" # enable control plane load balancing
  svc_enable: "true" # enable user plane load balancing
  vip_leaderelection: "true" # mandatory for L2 mode
  svc_election: "true"
```


```
microk8s helm repo add kube-vip https://kube-vip.github.io/helm-charts --force-update
```

```
microk8s helm install \
  kube-vip kube-vip/kube-vip \
  --namespace kube-system \
  --create-namespace \
  -f kube-vip.cnf
```

### Install cert-manager
Documentation: https://cert-manager.io/docs/installation/helm/
```
helm repo add jetstack https://charts.jetstack.io --force-update
```

```
helm upgrade --install \
  cert-manager jetstack/cert-manager \
  --version v1.17.0 \
  --namespace cert-manager \
  --create-namespace \
  --set crds.enabled=true
```

### INSTALL LONGHORN
> [!NOTE]
> This command is idempotent: if installed do upgrade else do install

Documentation: https://longhorn.io/docs/1.8.1/deploy/install/install-with-helm/
```
helm repo add longhorn https://charts.longhorn.io --force-update
```

```
helm upgrade --install \
  longhorn longhorn/longhorn \
  --version 1.8.1 \
  --namespace longhorn-system \
  --create-namespace \
  --repo https://charts.longhorn.io
  --set csi.kubeletRootDir="/var/snap/microk8s/common/var/lib/kubelet" \
  --set longhornUI.replicas=1 \
  --set ingress.enabled=true \
  --set ingress.host=longhorn.lab.rauhalat.org \
  --set ingress.tls=true \
  --set ingress.annotations."cert-manager\.io/cluster-issuer"=letsencrypt \
  --set ingress.tlsSecret=longhorn-ingress-tls
```

### INSTALL PORTAINER
> [!NOTE]
> This command is idempotent: if installed do upgrade else do install
Documentation: https://docs.portainer.io/start/install-ce/server/kubernetes/baremetal

```
helm repo add portainer https://portainer.github.io/k8s/ --force-update
```

```
helm upgrade --install \
  portainer portainer/portainer \
  --namespace portainer \
  --create-namespace \
  --repo https://portainer.github.io/k8s/ \
  --set service.type=ClusterIP \
  --set tls.force=true \
  --set image.tag=lts \
  --set ingress.enabled=true \
  --set ingress.ingressClassName=nginx \
  --set ingress.annotations."nginx\.ingress\.kubernetes\.io/backend-protocol"=HTTPS \
  --set ingress.annotations."cert-manager\.io/cluster-issuer"=letsencrypt \
  --set ingress.hosts\[0\].host=portainer.lab.rauhalat.org \
  --set ingress.hosts\[0\].paths\[0\].path="/" \
  --set ingress.tls\[0\].hosts\[0\]=portainer.lab.rauhalat.org \
  --set ingress.tls\[0\].secretName=portainer-ingress-tls
```

### Setup
Create alias for `microk8s kubectl`
```
sudo snap alias microk8s.kubectl kubectl
```

Create alias for `microk8s helm`
```
sudo snap alias microk8s.helm helm
```

## Upgrade microk8s
```
sudo microk8s kubectl drain node-a  --ignore-daemonsets
sudo snap refresh microk8s --channel 1.32/stable
sudo microk8s kubectl uncordon node-a
```

