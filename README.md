# HomeLab - MicroK8S

## Installation

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
microk8s helm repo add jetstack https://charts.jetstack.io --force-update
```

```
microk8s helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.16.2 \
  --set crds.enabled=true
```



### Setup

Add to `.bash_aliases`
```
alias kubectl='microk8s kubectl'
```
or
```
sudo snap alias microk8s.kubectl kubectl
```


## Upgrade microk8s
```
sudo microk8s kubectl drain node-a  --ignore-daemonsets
sudo snap refresh microk8s --channel 1.32/stable
sudo microk8s kubectl uncordon node-a
```
