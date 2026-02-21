<img src="../img/kubernetes-horizontal-color.png" />
																
   	KUBERNETES INSTALL UBUNTU		

Выключаем свап
```Shell
swapoff -a
```
```
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

 Перезагрузить и посмотреть что с swap

```Shell
swapon --show
```

modprobe overlay
modprobe br_netfilter

 Разрешим мостовый интерфейс - "дебильный английский язык"

```Shell
cat <<EOF | tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```


```Shell
cat <<EOF | tee /etc/sysctl.d/99-kubernetes-k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```Shell

Посмотрим параметры после перезагрузки

sysctl --system

Проверьте, сохранились ли изменения:
```Shell
sysctl net.ipv4.ip_forward
sysctl net.bridge.bridge-nf-call-ip6tables
sysctl net.bridge.bridge-nf-call-iptables
```

#
apt update
apt install -y containerd
apt install -y ca-certificates curl gpg socat conntrack ipset kmod vim


Включаем 
```Shell
sudo systemctl enable containerd && systemctl restart containerd
```

```Shell
sudo nano /etc/hosts
```

192.168.1.100 node-01.localdomain node-01

sudo hostnamectl set-hostname node-01


 Установка Kubernetes 	

```Shell
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
```Shell
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | tee /etc/apt/sources.list.d/kubernetes.list
```

```Shell
apt update
apt install kubelet kubeadm kubectl -y
apt-mark hold kubelet kubeadm kubectl
```

```Shell
systemctl enable --now kubelet
```

```Shell 
sudo kubeadm config images pull
```

```Shell 
sudo kubeadm init   --pod-network-cidr=10.244.0.0/16   --service-cidr=10.96.0.0/12
```

```Shell 
mkdir -p $HOME/.kube
```

```Shell 
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

```Shell 
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```




```Shell 
kubectl get pods --all-namespaces 
 ```



