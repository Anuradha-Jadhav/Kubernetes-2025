# Kubernetes-2025
Aws Master and Workder Node (Ubuntu server 24.04LTS)
64bit(x86)
t2medium 2vcpu 4GiB Memory
![image](https://github.com/user-attachments/assets/8456fecc-c734-4883-8c7e-7532b5320dcc)
![image](https://github.com/user-attachments/assets/4d921163-3019-4c20-9ad3-ab5280397543)
![image](https://github.com/user-attachments/assets/cc2b3b73-2cb4-48a0-ae5b-a4d9eaecd0c1)

# Kubernetes Setup and Nginx Deployment

## Prerequisites
Ensure your system is up to date:
```sh
sudo apt-get update
sudo apt-get upgrade -y
```

## Disable Swap
```sh
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

## Configure Kernel Parameters
Load necessary modules:
```sh
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

Enable networking settings:
```sh
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

## Install Containerd Runtime
```sh
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt update
sudo apt install -y containerd.io

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd
```

## Install Kubernetes Components
```sh
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### Install Kubelet, Kubeadm, and Kubectl
```sh
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## Initialize Kubernetes Master Node
```sh
sudo kubeadm init
```

Set global Kubeconfig variable:
```sh
export KUBECONFIG=/etc/kubernetes/admin.conf
```

Set up the cluster:
```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Join Worker Nodes
Generate a token and join command:
```sh
kubeadm token create --print-join-command
```
Run the generated command on the worker node with `sudo`.

## Configure Networking (Calico) Saved in seperate file in this repo
Create `calico.yaml` and apply it:
```sh
sudo chmod 764 /etc/kubernetes/admin.conf
kubectl apply -f calico.yaml
```

---

# Nginx Deployment with Custom HTML Page

Deploy Nginx:
```sh

kubectl create deployment nginx --image=nginx:latest --port=80
kubectl expose deployment nginx --type=NodePort --port=80
kubectl get svc
```

Access Nginx Port may vary:
```sh
curl -o http://publicipof workernode:<pod port>
curl -o http://54.171.60.51:30355
curl http://54.171.60.51:30355
```

Manage Nginx pods:
```sh
kubectl get pods
kubectl exec -it nginx-7c79c4bf97-zjd7v -- /bin/sh
```

Modify the index page:
```sh
vi index.html
```

Copy the modified page to the pod:
```sh
kubectl cp index.html nginx-7c79c4bf97-zjd7v:/usr/share/nginx/html/
```

Verify the changes:
```sh
curl http://54.171.60.51:30355
```






