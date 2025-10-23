# Installation d’un cluster Kubernetes v1.34 sur Ubuntu 22.04

Ce guide explique comment installer un **cluster Kubernetes v1.34** composé d’un **nœud master** et d’un **nœud worker** à l’aide de `kubeadm`, sur Ubuntu 22.04.

---

## 1. Architecture du cluster

| Rôle | Nom d’hôte | CPU | RAM | IP (exemple) |
|------|-------------|-----|-----|--------------|
| Master | `k8s-master-o1` | 4 vCPU | 8 Go | 192.168.122.225 |
| Worker | `k8s-worker-01` | 2 vCPU | 4 Go | 192.168.122.113|
| Worker | `k8s-worker-02` | 2 vCPU | 4 Go | 192.168.122.156 |

> 💡 Assurez-vous que chaque machine possède une adresse IP fixe et que le nom d’hôte est configuré correctement.

---

## 2. Configuration système de base

Exécuter ces commandes **sur tous les nœuds** :

```bash
sudo apt update && sudo apt upgrade -y
sudo hostnamectl set-hostname <nom_du_noeud>

# Désactiver le swap
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# Activer les modules réseau nécessaires
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Configurer les paramètres réseau
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```
## 3. Installation de Containerd
```bash
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# Activer SystemdCgroup
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd
```
## 4. Installation de kubeadm, kubelet et kubectl (v1.34)
```bash 
sudo apt install -y apt-transport-https ca-certificates curl gpg

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
## 5. Initialisation du Master
Sur le master uniquement :
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
> 💡 À la fin de l’installation, une commande kubeadm join ... s’affichera : gardez-la précieusement pour ajouter les nœuds workers.

## Configurer kubectl pour l’utilisateur courant :
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
## 6. Installation du plugin réseau (CNI) 
nous avons fais le choix de Flannel
```bash
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```
## 7. Ajouter le nœud Worker

> 💡 Avant d'ajouter les workers assurer vous d'abord que le master est pret (ready) comme le montre la capture ci-dessous

```bash
kubectl get nodes
```

![Plot](/images/master-ready.png)
Sur le worker, exécuter la commande affichée à la fin de kubeadm init, par exemple :
```bash
kubeadm join 192.168.122.225:6443 --token vtsaj3.5umh3cbe6uog103i \
	--discovery-token-ca-cert-hash sha256:fea096d22ca49cdf11d412055be2a7cba2c4084b3f00f00f3944d3ec27c1b772 
```
![Plot](/images/join-node.png)
## 8. Vérifier le cluster
```bash
kubectl get nodes
kubectl get pods -A
```
![Plot](/images/cluster-ready.png)

## les infos du cluster:
![Plot](/images/info.png)


-----
Constantin S.E BASSENE
