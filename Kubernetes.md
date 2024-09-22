# Kubernetes 🛞

<br>

---

## Sommaire

1. Déploiement du homelab
2. 

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
    kub.vm.provision "docker"
    kub.vm.box_url = "bento/ubuntu-22.04"

    kub.vm.network :private_network, ip: "10.10.1.101"

    kub.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      v.customize ["modifyvm", :id, "--memory", 1024]
      v.customize ["modifyvm", :id, "--name", "kmaster"]
      v.customize ["modifyvm", :id, "--cpus", "1"]
    end
  end

  config.vm.define "knode1" do |knode|
    knode.vm.box = "bento/ubuntu-22.04"
    knode.vm.hostname = 'knode1'
    knode.vm.provision "docker"
    knode.vm.box_url = "bento/ubuntu-22.04"
    knode.vm.network :private_network, ip: "10.10.1.102"
    knode.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      v.customize ["modifyvm", :id, "--memory", 1024]
      v.customize ["modifyvm", :id, "--name", "knode1"]
      v.customize ["modifyvm", :id, "--cpus", "1"]
    end
  end
  config.vm.define "knode2" do |knode|
    knode.vm.box = "bento/ubuntu-22.04"
    knode.vm.hostname = 'knode2'
    knode.vm.provision "docker"
    knode.vm.box_url = "bento/ubuntu-22.04"
    knode.vm.network :private_network, ip: "10.10.1.103"
    knode.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      v.customize ["modifyvm", :id, "--memory", 1024]
      v.customize ["modifyvm", :id, "--name", "knode2"]
      v.customize ["modifyvm", :id, "--cpus", "1"]
    end
  end
end
```

Installation du pré-requis **Docker**

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

Lancement au démarrage de kubelet

```
systemctl enable kubelet
```
