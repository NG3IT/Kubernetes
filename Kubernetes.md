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

    kub.vm.network :private_network, ip: "10.10.1.1"

    kub.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      v.customize ["modifyvm", :id, "--memory", 1024]
      v.customize ["modifyvm", :id, "--name", "kmaster"]
      v.customize ["modifyvm", :id, "--cpus", "1"]
    end
  end

  config.vm.define "knode" do |knode|
    knode.vm.box = "bento/ubuntu-22.04"
    knode.vm.hostname = 'knode'
    knode.vm.provision "docker"
    knode.vm.box_url = "bento/ubuntu-22.04"
    knode.vm.network :private_network, ip: "10.10.1.2"
    knode.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      v.customize ["modifyvm", :id, "--memory", 1024]
      v.customize ["modifyvm", :id, "--name", "knode"]
      v.customize ["modifyvm", :id, "--cpus", "1"]
    end
  end
  config.vm.define "knode2" do |knode|
    knode.vm.box = "bento/ubuntu-22.04"
    knode.vm.hostname = 'knode2'
    knode.vm.provision "docker"
    knode.vm.box_url = "bento/ubuntu-22.04"
    knode.vm.network :private_network, ip: "10.10.1.3"
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
swapoff -a

```

Installation du **repo Kubernetes**

```
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
sudo add-apt-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

