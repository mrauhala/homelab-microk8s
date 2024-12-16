# HomeLab - MicroK8S

## Installation

### Setup

Add to *.bash_aliases*
```
alias kubectl='microk8s kubectl'
```


## Upgrade microk8s
```
sudo microk8s kubectl drain node-a  --ignore-daemonsets
sudo snap refresh microk8s --channel 1.32/stable
sudo microk8s kubectl uncordon node-a
```
