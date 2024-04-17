# Installer Kubernetes sur Ubuntu 22.04 
# 1) Mettre a jour le systeme
$ sudo apt update && sudo apt upgrade -y

# 2) Installer le runtime docker
$ sudo apt install docker.io -y

# 3) Désactiver les partitions swap
$ sudo swapoff -a
$ sed -e '/swap/s/^/#/g' -i /etc/fstab

# 4) Installer les composants de kubernetes: kubeadm, kubelet et kubectl
$ sudo apt update && sudo apt install -y apt-transport-https
$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
$ sudo apt update
$ sudo apt install -y kubelet kubeadm kubectl
$ sudo apt-mark hold kubelet kubeadm kubectl (optionnel)

# 5) Activer le service kubelet
$ systemctl enable --now kubelet.service

# 6) Initialiser le cluster kubernetes ( a faire uniquement sur le node master ou controleur)
$ sudo kubeadm init --pod-network-cidr=192.168.0.0/16

# Cette commande initialisera le cluster Kubernetes et créera un nouveau fichier de configuration dans /etc/kubernetes/admin.conf. Copiez ce fichier dans votre répertoire personnel à l'aide de la commande suivante :

$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 7) Joindre les workers au cluster ( a faire uniquement sur les node worker )
$ $ sudo kubeadm join <master-ip>:<master-port> --token <token> --discovery-token-ca-cert-hash <hash>





