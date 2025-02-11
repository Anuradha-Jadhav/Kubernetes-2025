# Kubernetes-2025
Aws Master and Workder Node (Ubuntu server 24.04LTS)
64bit(x86)
t2medium 2vcpu 4GiB Memory
![image](https://github.com/user-attachments/assets/8456fecc-c734-4883-8c7e-7532b5320dcc)
![image](https://github.com/user-attachments/assets/4d921163-3019-4c20-9ad3-ab5280397543)
![image](https://github.com/user-attachments/assets/cc2b3b73-2cb4-48a0-ae5b-a4d9eaecd0c1)

# Configure Kubernetes Cluster With Kubeadm on Ubuntu

## 1. Take 3 Ubuntu servers (I'Ve taken AWS Ubuntu 3 servers)
```
  One node for master and other two workers
```

## 2. Repeat these steps on all three nodes

  * ### Install Docker on all nodes
  ```
      sudo apt update
      sudo apt install -y docker.io
      sudo systemctl start docker
      sudo systemctl enable docker
  ```
  
  * ### Adding current user to docker group (Optional)
      ```
        sudo usermod -aG docker $USER
        newgrp docker
      ```
  * ### Disable swap memory
  
      ```
        sudo swapoff -a
      
      ```
    
  * ### Command to update fstab so that swap remains disabled after a reboot
  
      ```
        sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
      ```
    
  * ### Disable AppArmor and set SELinux in permissive mode (if applicable)
  ```
        sudo systemctl stop apparmor
        sudo systemctl disable apparmor
  ```
  
  * ### Install kubeadm, kubectl, kubelet on all three nodes
      
```
sudo apt install -y apt-transport-https ca-certificates curl
   curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo tee /etc/apt/keyrings/kubernetes-apt-keyring.asc > /dev/null
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.asc] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
```
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```     
## 3. Initialize Kubernetes cluster on Master node (Run this only on master)
    ```
    sudo kubeadm init
    ```
        
## 4. Do the following setup to start using Kubernetes cluster (Run this only on master)
  ```
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```
## 5. Add pod network add-ons (Run this only on master)
  
    ```
    kubectl apply -f https://github.com/weaveworks/weave/releases/latest/download/weave-daemonset-k8s.yaml
    
    ```
        
  ## 6. Take note of kubeadm command and run on all workers
  
  ```
    Run this command (replace with your actual command) on every worker node to join the cluster:
  
    sudo kubeadm join <master-ip>:6443 --token <token> \
      --discovery-token-ca-cert-hash sha256:<hash>
  ```

  ## 7. Get list of nodes in the cluster (Run this on master)
  
  ```
  kubectl get nodes
  ```




