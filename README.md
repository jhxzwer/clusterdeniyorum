1.29 versiyonu ile kurulum gerçekleştirdim. İlerleyen dönemlerde upgrade işlemi gerçekleştirmek için alt versiyon kullandım.
--------------------------------------------------------------------------------------------
https://v1-29.docs.kubernetes.io/docs/setup/production-environment/container-runtimes/

Kernel modunun aktifleştirilmesi ve sysctl konfigurasyonu :

-------------------------------------------------------------------------------------------
sudo modprobe overlay
sudo modprobe br_netfilter
-------------------------------------------------------------------------------------------
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
-------------------------------------------------------------------------------------------
sudo sysctl --system
-------------------------------------------------------------------------------------------

Container Runtime Kurulumu :

-------------------------------------------------------------------------------------------
sudo tee /etc/modules-load.d/k8s.conf <<EOF
overlay
br_netfilter
EOF
-------------------------------------------------------------------------------------------
sudo modprobe overlay
sudo modprobe br_netfilter
-------------------------------------------------------------------------------------------
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
-------------------------------------------------------------------------------------------
sudo sysctl --system
-------------------------------------------------------------------------------------------
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
-------------------------------------------------------------------------------------------
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
-------------------------------------------------------------------------------------------
sudo apt update
sudo apt install -y containerd.io
-------------------------------------------------------------------------------------------
sudo su -
mkdir -p /etc/containerd
containerd config default>/etc/containerd/config.toml
-------------------------------------------------------------------------------------------
sudo systemctl restart containerd
sudo systemctl enable containerd
sudo systemctl status containerd
-------------------------------------------------------------------------------------------
cat /etc/containerd/config.toml | grep SystemdCgroup
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
-------------------------------------------------------------------------------------------
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
-------------------------------------------------------------------------------------------
sudo sysctl --system
-------------------------------------------------------------------------------------------
lsmod | grep br_netfilter
lsmod | grep overlay
-------------------------------------------------------------------------------------------
Verify; 

sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
--------------------------------------------------------------------------------------------
Swap'in kapatılması : 

Swap disable ;
vi /etc/fstab
sudo swapoff -a
sudo mount -a
free -h
-------------------------------------------------------------------------------------------

Kubeadm,Kubectl,Kubelet kurulumları ; 
https://v1-29.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

-------------------------------------------------------------------------------------------
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
--------------------------------------------------------------------------------------------

Şimdi cluster'ımızı init edelim.

--------------------------------------------------------------------------------------------

sudo kubeadm init --control-plane-endpoint="kubernetes.maraz:6443" --apiserver-advertise-address=192.168.201.10 --node-name master --pod-network-cidr=10.10.0.0/16










