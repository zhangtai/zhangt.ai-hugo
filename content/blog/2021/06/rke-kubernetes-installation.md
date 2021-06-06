---
title: RKE Kubernetes Installation
date: 2021-06-06T12:20:48+08:00
---

[Official Docs](https://rancher.com/docs/rke/latest/en/installation/)

I have a always on Proxmox server at home with 32GB ECC RAM and 36 logical cores, so instead of install tools like minikube running on single machine I considered to install a more production like k8s cluster with multiple workers on each VM.

## Provisioning VMs

With [terraform proxmox provider](https://github.com/Telmate/terraform-provider-proxmox) I will spin up 5 VMs with static IP addresses, based on the ubuntu cloud-init image with docker installed.

## Install RKE cluster

Create `cluster.yml` file as below and run `rke up`

```yaml
nodes:
- address: 192.168.3.30
  user: rancher
  role:
  - controlplane
  - etcd
- address: 192.168.3.31
  user: rancher
  role:
  - worker
- address: 192.168.3.32
  user: rancher
  role:
  - worker
- address: 192.168.3.33
  user: rancher
  role:
  - worker
- address: 192.168.3.34
  user: rancher
  role:
  - worker
```

## Install Nginx Ingress with helm

```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
# The default image registry is k8s.gcr.io which is not accessible in China
# Found an alternative on docker hub, hence adding below params
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --set controller.image.registry=docker.io \
  --set controller.image.image=pollyduan/ingress-nginx-controller \
  --set controller.image.digest=''

kubectl --namespace default get services -o wide -w ingress-nginx-controller
```

## Install local pv provisioner

```sh
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```
