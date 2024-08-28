1.29 versiyonu ile kurulum gerçekleştirdim.
--------------------------------------------------------------------------------------------
https://v1-29.docs.kubernetes.io/docs/setup/production-environment/container-runtimes/

Sunucuya gui yükledim. 

sudo apt install slim

sudo apt install ubuntu-desktop
https://phoenixnap.com/kb/how-to-install-a-gui-on-ubuntu

-------------------------------------------------------------------------------------------
Container runtime için ön gereksinim konfigurasyonu
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
sudo modprobe overlay
sudo modprobe br_netfilter
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
# Apply sysctl params without reboot
sudo sysctl --system
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
lsmod | grep br_netfilter
lsmod | grep overlay
-------------------------------------------------------------------------------------------
Verify; 

sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
--------------------------------------------------------------------------------------------
Swap disable ;
vi /etc/fstab
sudo swapoff -a
# Confirm setting is correct
sudo mount -a
free -h
--------------------------------------------------------------------------------------------
# Reload configs
sudo sysctl --system
# Install required packages
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
# Add Docker repo
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
# Install containerd
sudo apt update
sudo apt install -y containerd.io
# Configure containerd and start service
sudo su -
mkdir -p /etc/containerd
containerd config default>/etc/containerd/config.toml
# restart containerd
sudo systemctl restart containerd
sudo systemctl enable containerd
sudo systemctl status containerd
# To use the systemd cgroup driver, set plugins.cri.systemd_cgroup = true 
cat /etc/containerd/config.toml | grep SystemdCgroup
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
-------------------------------------------------------------------------------------------
Alternatif ; ; 
Containerd kurulumu ;  https://github.com/containerd/containerd/blob/main/docs/getting-started.md
From APT | - >  https://docs.docker.com/engine/install/ubuntu/
sudo apt-get install containerd.io 
Containerd konfigini resetlemek için ; containerd config default > /etc/containerd/config.toml
/etc/containerd/config.toml dosyasının içerisindeki "cri" kısmını kaldırdım, sonra aşağıdaki satırları ekledim. (containerd çalışması için bu konfigurasyonları yaptım)
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
SystemdCgroup = true
sudo systemctl restart containerd
-----------------------------------------------------------------------------------------------

Kubeadm,Kubectl,Kubelet kurulumları ; 

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










