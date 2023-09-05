# belajark8 - Instalasi k8s menggunakan kubeadm

STEP01
#install runtime container
cat > /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

swapoff -a
rm /swap.img
vi /etc/fstab #lalu comment pada swap
#/swap.img       none    swap    sw      0       0

cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system

apt-get update && apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
	
apt-get update && apt-get install -y containerd.io

mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

systemctl restart containerd

STEP02
#Install Kubeadm
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update

#install specific version
apt list -a kubeadm
sudo apt-get install -y kubelet=1.26.0-00 kubeadm=1.26.0-00 kubectl=1.26.0-00

#install latest version
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

sudo systemctl daemon-reload
sudo systemctl restart kubelet

STEP03
#Kubeadm init
kubeadm init --control-plane-endpoint="10.128.0.4:6443" --upload-certs --apiserver-advertise-address=10.128.0.4 --pod-network-cidr=192.168.0.0/16

atau, tambahkan --cri-socket apabila ada error multiple runtime container

--cri-socket unix:///var/run/containerd/containerd.sock

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

#STEP04
###Setup add on network untuk service
curl https://raw.githubusercontent.com/projectcalico/calico/v3.24.5/manifests/canal.yaml -O
kubectl -n kube-system apply -f canal.yaml
