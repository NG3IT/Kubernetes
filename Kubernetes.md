# Kubernetes 🛞

<br>

---

## Sommaire

1. Déploiement du homelab
2. Installation et configuration de Kubernetes
3. Troubleshooting

<br>

---

## Déploiement du homelab 🧪

Pour déployer rapidement et simplement un homelab K8s avec plusieurs nodes, on peut utiliser Vagrant.

Fichier **Vagrant**

```txt
Vagrant.configure("2") do |config|
  config.vm.define "kmaster" do |kub|
    kub.vm.box = "bento/ubuntu-22.04"
    kub.vm.hostname = 'kmaster'
    kub.vm.box_url = "bento/ubuntu-22.04"

    kub.vm.network :private_network, ip: "10.10.1.101"

    kub.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      v.customize ["modifyvm", :id, "--memory", 2048]
      v.customize ["modifyvm", :id, "--name", "kmaster"]
      v.customize ["modifyvm", :id, "--cpus", "2"]
    end
  end

  config.vm.define "knode" do |knode|
    knode.vm.box = "bento/ubuntu-22.04"
    knode.vm.hostname = 'knode'
    knode.vm.box_url = "bento/ubuntu-22.04"
    knode.vm.network :private_network, ip: "10.10.1.102"
    knode.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      v.customize ["modifyvm", :id, "--memory", 2048]
      v.customize ["modifyvm", :id, "--name", "knode"]
      v.customize ["modifyvm", :id, "--cpus", "2"]
    end
  end
end
```

Installation du pré-requis **Docker** (si pas de containerd)

```
curl -fsSL https://get.docker.com | sh;
sudo usermod -aG docker $USER

groupadd -g 500000 dockremap && 
groupadd -g 501000 dockremap-user && 
useradd -u 500000 -g dockremap -s /bin/false dockremap && 
useradd -u 501000 -g dockremap-user -s /bin/false dockremap-user

echo "dockremap:500000:65536" >> /etc/subuid && 
echo "dockremap:500000:65536" >>/etc/subgid

echo "
  {
   \"userns-remap\": \"default\"
  }
" > /etc/docker/daemon.json

systemctl daemon-reload && systemctl restart docker
```

Désactivation du **swap**

```
# Swap -> OFF
swapoff -a

# Désactivation du point de montage de la SWAP
vim /etc/fstab
#/swap.img      none    swap    sw      0       0
```


<br>

--- 

## Installation et configuration de Kubernetes 🔧

Ajout du **repo Kubernetes**

```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
```

Installation des **binaires K8s**

```
sudo apt-get install -y kubelet kubeadm kubectl kubernetes-cni
```

Lancement au **démarrage du service kubelet**

```
systemctl enable kubelet
```

Initilisation du **réseau de Kubernetes** depuis le **master**

```
kubeadm init --apiserver-advertise-address=<ip_master_k8s> --node-name $HOSTNAME --pod-network-cidr=10.244.0.0/16
kubeadm init --apiserver-advertise-address=10.10.1.101 --node-name $HOSTNAME --pod-network-cidr=10.244.0.0/16
```

Création du **fichier de configuraiton** de Kubernetes

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.cnf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Mise en place du **réseau interne (Flannel)**

```
# Ajout du pod de gestion du réseau interne
sysctl net.bridge.bridge-nf-call-iptables=1

# Application du fichier de configuration
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

Vérification de l'états des pods system

```
kubectl get pods --all-namespaces
kubectl get nodes
```

<br>

---

## 3. Troubleshoot

**⚠️ Attention ⚠️
Les 2 erreurs ci-dessous sont liés à un sous-dimensionnement du master K8s. Il est possible de bypassé cette restriction, cependant, le cluster risque de ne pas fonctionner par la suite avec des pods en status "CrashLoopBackOff" par exemple**

3.1 ERROR NumCPU

**ERROR NumCPU** au moment de l'initialisation du master -> Pour fonctionner correctement Kubernetes a besoin de tourner sur un serveur qui dispose d'au moins deux vCPU.

```
[ERROR NumCPU]: the number of available CPUs 1 is less than the required 2
```

**Solution 1** -> Modifier la configuration matériel pour avoir le nombre de vCPU requis

**Solution 2** -> Ignorer l'erreur. **A ne pas faire dans un environnement de production**

```
# Ajout du flag --ignore-preflight-errors=NumCPU
kubeadm init --apiserver-advertise-address=10.10.1.101 --node-name $HOSTNAME --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=NumCPU
```

---

3.2 ERROR Mem

**ERROR Mem** au moment de l'initialisation du master -> Pour fonctionner correctement Kubernetes a besoin de tourner sur un serveur qui dispose d'au moins deux Go de RAM.

```
[ERROR Mem]: the system RAM (957 MB) is less than the minimum 1700 MB
```

**Solution 1** -> Modifier la configuration matériel pour avoir le nombre de RAM requis

**Solution 2** -> Ignorer l'erreur. **A ne pas faire dans un environnement de production**

```
# Ajout du flag --ignore-preflight-errors=Mem
kubeadm init --apiserver-advertise-address=10.10.1.101 --node-name $HOSTNAME --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=Mem
```

---

3.3 ERROR CRI

**ERROR Mem** au moment de l'initialisation du master -> Pour fonctionner correctement Kubernetes a besoin d'un élément qui va géré les container (Container Runtime Interface).

```
[ERROR CRI]: container runtime is not running
```

**Solution 1** -> Modifier la configuration de containerd (dans le cas d'un Ubuntu supérieur à la version 22.04) 

```
# Désactivation des "disabled_plugins"
vim /etc/containerd/config.toml
#disabled_plugins = ["cri"]

sudo systemctl restart containerd 
```

---

3.4 sysctl: cannot stat /proc/sys/net/bridge/bridge-nf-call-iptables

Au moment de l'activation du filtrage des paquets bridge via iptables, il est possible d'avoir cette erreur :

```
sysctl: cannot stat /proc/sys/net/bridge/bridge-nf-call-iptables: No such file or directory
```

Cette erreur est probablement du au fait que le module "br_netfilter" n'est pas chargé au niveau du noyau.

```
# Activatgion du module br_netfilter
sudo modprobe br_netfilter

# Vérification du chargement du module
lsmod | grep br_netfilter
```

---

3.5 Pod en status "CrashLoopBackOff"

Lorsque l'on liste les pods de tous les namespaces par exemple, on constate un pod en status "CrashLoopBackOff"

```
kube-system    kube-controller-manager-kmaster   0/1     CrashLoopBackOff   17 (60s ago)     156m
```
