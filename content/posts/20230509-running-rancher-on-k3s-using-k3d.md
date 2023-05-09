---
title: "Running Rancher on K3S using K3D"
date: 2023-05-09T10:07:26+02:00
draft: false
tags: ["kubernetes", "rancher", "k3d", "docker"]
categories: ["infrastructure"] 
---

## Requirements

- [Colima](https://github.com/abiosoft/colima) installed
  - When using Colima make sure to point to the correct docker socket using an export  
    `export DOCKER_HOST="unix://${HOME}/.colima/default/docker.sock`
- [K3D](https://k3d.io/) installed
- [Helm](https://helm.sh/) installed 

## Steps
```
# start colima with 4 CPU and 8GB memory
colima start --cpu 4 --memory 8

# add the correct Helm repositories
echo "Adding Helm repos" \
&& helm repo add rancher-latest https://releases.rancher.com/server-charts/latest \
&& helm repo add rancher-stable https://releases.rancher.com/server-charts/stable \
&& helm repo add jetstack https://charts.jetstack.io \
&& helm repo update

# create a cluster where rancher will be installed (name: rmaster -> this will become k3d-rmaster)
## - port 80/443 on localhost will be forwarded to 80/443 in the cluster/load balancer
## - the number of servers and agents are specified explicitly but they are the defaults!
##   servers: the number of servers that will be run
##   agents: the number of worker nodes
k3d cluster create rmaster --port 80:80@loadbalancer --port 443:443@loadbalancer --servers 1 --agents 0

# change to the correct Kubernetes context
## actually not required as k3d will switch to the just created context
kubectl config set-context k3d-rmaster

# install Cert Manager
kubectl create namespace cert-manager \
&& helm install cert-manager jetstack/cert-manager \
    --namespace cert-manager \
    --version v1.11.1 \
    --set installCRDs=true --debug

# verify if cert-manager is running
kubectl get pods --namespace cert-manager

# install Rancher (this can take a while! So dont get scared when you see pods in error state!)
## - replicas: the number of rancher replicas
## - hostname: set the hostname to your correct ip; for this we are using https://nip.io
##             an alternative could be using ngrok or manually update your hosts file!
## - bootstrapPassword: The password for login into the rancher web ui
## - global.cattle.psp.enabled: We disable pod security policies; we need to harden later on!
kubectl create namespace cattle-system \
&& helm install rancher rancher-latest/rancher \
    --namespace cattle-system \
    --set replicas=1 \
    --set hostname=192-168-68-107.nip.io \
    --set global.cattle.psp.enabled=false \
    --set bootstrapPassword=welcome123 \
    --debug

# verify if rancher is running
kubectl get pods --namespace cattle-system

# create client clusters
k3d cluster create rclient1 # (name: rclient1 -> this will become k3d-rclient1)
k3d cluster create rclient2 # (name: rclient2 -> this will become k3d-rclient2)

# import the client clusters via the web interface!
## make sure to pick the correct host; see the rancher install
## this will give an unsecure connection; no need to worry just proceed in your browser
http://192-168-68-107.nip.io
```

## Resources

- [GitHub Gist - Rancher Workshop](https://gist.github.com/rafi/d4440661e7de208009701ca3627caa1d)
- [GitHub Gist - Script to install rancher on your macOS machine](https://gist.github.com/jwsy/5634d17e133d7fdc3045c3a2a18b652a)
