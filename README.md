# Kubernetes Learning Series: Workshop

## Prerequistes
- [K|X|L]Ubuntu 22.04 LTS or Using Ubuntu Server 22.04 LTS 
- Use of **K3S**
- Established SSH Connection to Server that K3S will live on. 


### (Optional) Setting up Fish Shell 
> Fish is a shell for Linux, MacOS or WSL2 that includes autocompletion and information for commands. 

1. Install Fish 
    ```bash
    sudo apt update && sudo apt install -y fish
    ```
1. Enter Fish in the  
    ```bash 
    fish
    ```
1. Set Fish as your default shell 
    ```bash
    chsh -s /usr/bin/fish
    ```
1. Check if config file exists, if not create one
    ```bash 
    nano ~/.config/fish/config.fish #This will throw a error if file doesn't exist
    ```
    ```bash
    mkdir -p ~/.config/fish

    nano ~/.config/fish/config.fish
    ```
## Setting up Variables

### Setup Domain
```bash
export DOMAIN=example.com
```

### Setup Email
```bash
export EMAIL=name@example.com
```

To make these changes permanent in the terminal make the do the following. 
> In Linux the tilda `~` references the home directory aka `/home/$USER`

#### bash
```bash 
nano ~/.profile
#Add the following line to the bottom of the file: 
    export DOMAIN=example.com
    export EMAIL=name@example.com
    export KUBECONFIG=/etc/rancher/k3s/k3s.yaml 
```

#### fish 
```bash
nano ~/.config/fish/config.fish
#Add the following lines to the bottom of the file: 
    set -gx DOMAIN example.com
    set -gx EMAIL name@example.com\
    set -gx KUBECONFIG /etc/rancher/k3s/k3s.yaml 
```

### Install pieces needed for Longhorn and Wireguard
```bash
sudo apt update && \
sudo apt upgrade -y && \ 
sudo apt install open-iscsi -y && \ 
sudo apt install wireguard -y
```

### Setup Arkade, Heml and Kubectl autocomplete
The following section installs the Arkade and then uses Arkade to install the other tools. Helm will be used to download per configured Kubernetes Apps/environments and kubectl is the management tool. 

```bash
curl -sLS https://dl.get-arkade.dev | sh && \ 
sudo cp arkade /usr/local/bin/arkade && \ 
sudo cp arkade /usr/local/bin/arkade && \ 
arkade get helm && \ 
sudo cp ~/.arkade/bin/helm /usr/local/bin/ && \ 
arkade get kubectl && \ 
sudo mv ~/.arkade/bin/kubectl /usr/local/bin/ 
```
### Add kubectl autocompletion
bash
```bash
source <(kubectl completion bash) && \ 
echo 'source <(kubectl completion bash)' >> ~/.bashrc
```

fish 
```bash
echo "kubectl completion fish | source" >> ~/.config/fish/config.fish
```

## Installing k3s and setting up main agent. 
> In this guide we are not setting a single node. It is suggested to have multiple worker nodes to have a high availablity cluster. 

Normally k3s installations come with Traefik installed, while it normally works out of the box, we will switch to using the Nginx Ingress controller.


```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.23.5+k3s1 sh -s server \ 
--cluster-init \ 
--flannel-backend=wireguard \ 
--write-kubeconfig-mode 644 \
--no-deploy traefik
```

bash 
```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml 
```

fish 
```bash 
set -gx KUBECONFIG /etc/rancher/k3s/k3s.yaml
```

### Install the Nginx Controller
### Create the namespace for ingress-nginx
```
kubectl create namespace ingress-nginx
```

### Add nginx repository to helm
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

### Install nginx ingress controller

```bash 
helm install ingress-nginx ingress-nginx/ingress-nginx --version="4.0.17" \
    --set rbac.create=true \
    --set controller.kind=DaemonSet \
    --set controller.service.type=LoadBalancer \
    --set controller.service.externalTrafficPolicy=Local \
    --set controller.publishService.enabled=true \
    --set defaultBackend.enabled=true \
    --set enable-prometheus-metrics=true \
    --namespace ingress-nginx
```

## Download and Install Lens 
On your Windows Installation: 

Visit the [Lens Website](https://k8slens.dev/) to install Lens (free for personal use) or use the following guide to install [OpenLens](https://blog.devgenius.io/is-it-time-to-migrate-from-lens-to-openlens-75496e5758d8)

Connect to Kubernetes Server
Run the following command to get the k3s configuration
```bash
cat /etc/rancher/k3s/k3s.yaml
```
This is the default location for K3s install

Copy into Lens Kubeconfig edit section
Change Ip address from localhost to external IP
```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0...
    server: [Change This Here to IP address]:6443
  name: default
contexts:
```
