---
categories:
    - infrastructure
date: 2025-01-24T17:25:36Z
description: Comprehensive guide to deploy Kubernetes cluster on KVM/Ubuntu, covering cloud-init automation and containerd configuration
keywords: kubernetes deployment, kvm virtualization, ubuntu cluster, containerd runtime, production kubernetes
tags:
    - kubernetes
    - kvm
    - ubuntu
    - containerd
    - cloud-init
    - k8s
title: 'Kubernetes Cluster Deployment: KVM-based Production Setup on Ubuntu'
---



# KVM Deployment of Kubernetes Cluster Operations Manual

## Preface

This manual provides detailed steps for deploying a complete Kubernetes cluster using KVM on Ubuntu 22.04 LTS system.

## Environment Requirements

### Hardware Configuration

- CPU: Supports hardware virtualization (Intel VT-x or AMD-V must be enabled)
- Memory: Minimum 8GB (2GB per node)
- Storage: Minimum 120GB free space
- Network: Stable network connection supporting virtual bridges

### Software Versions

- OS: Ubuntu 22.04 LTS
- Kubernetes: v1.32.1
- Container Runtime: containerd (latest stable version)

## 1. Basic Environment Preparation

### 1.1 Environment Variable Configuration

```bash
# Create working directory structure
export X_DIR="/opt/k8s"
export X_DOWNLOAD_DIR="${X_DIR}/download"
export X_IMG_DIR="${X_DIR}/images"
export X_CFG_DIR="${X_DIR}/configs"
export X_NET="k8s-net"
export X_OS_VARIANT="ubuntu22.04"
export X_BASE_IMG="${X_DOWNLOAD_DIR}/jammy-server-cloudimg-amd64.img"
export QCOW2_URL="https://cloud-images.ubuntu.com/jammy/releases/jammy/release/jammy-server-cloudimg-amd64.img"

# Initialize directories
sudo mkdir -p $X_DOWNLOAD_DIR $X_IMG_DIR $X_CFG_DIR
sudo chown -R $USER:$USER $X_DIR

# Download base image
[[ ! -f ${X_BASE_IMG} ]] && curl -fsSL -o "${X_BASE_IMG}" "${QCOW2_URL}"

# Clean
sudo virsh net-destroy ${X_NET} 2>/dev/null
sudo virsh net-undefine ${X_NET} 2>/dev/null
sudo virsh destroy k8s-cp-01 2>/dev/null
sudo virsh destroy k8s-worker-01 2>/dev/null
sudo virsh destroy k8s-worker-02 2>/dev/null
sudo virsh undefine k8s-cp-01 --remove-all-storage 2>/dev/null
sudo virsh undefine k8s-worker-01 --remove-all-storage 2>/dev/null
sudo virsh undefine k8s-worker-02 --remove-all-storage 2>/dev/null
sudo rm -rf ${X_CFG_DIR}/* ${X_IMG_DIR}/*
```

### 1.2 Install Required Components

```bash
# System update
sudo apt-get update && sudo apt-get upgrade -y

# Install KVM and related tools
sudo apt-get install -y \
    qemu-system-x86 \
    libvirt-daemon-system \
    libvirt-clients \
    bridge-utils \
    virt-manager \
    virtinst \
    cloud-image-utils \
    wget \
    curl

# Verify KVM installation
kvm-ok
sudo systemctl enable --now libvirtd
sudo systemctl status libvirtd
lsmod | grep kvm
```

## 2. Virtual Network Configuration

### 2.1 Create Dedicated Network

```bash
cat << EOF > ${X_CFG_DIR}/k8s-network.xml
<network>
  <name>${X_NET}</name>
  <forward mode="nat"/>
  <bridge name="virbr-k8s" stp="on" delay="0"/>
  <ip address="192.168.122.1" netmask="255.255.255.0">
    <dhcp>
      <range start="192.168.122.2" end="192.168.122.254"/>
    </dhcp>
  </ip>
</network>
EOF

# Deploy network
sudo virsh net-define ${X_CFG_DIR}/k8s-network.xml
sudo virsh net-start ${X_NET}
sudo virsh net-autostart ${X_NET}
sudo virsh net-list --all  # Verify network status
```

## 3. Virtual Machine Deployment

### 3.1 Create Cloud-Init Configuration File

```bash
# Generate SSH key (if not exists)
[[ ! -f ~/.ssh/id_ed25519 ]] && ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""

# Create cloud-init template
cat << EOF > ${X_CFG_DIR}/cloud-init.yml
#cloud-config

# Basic system configuration
hostname: myhost
fqdn: myhost.example.com

# User setup configuration
users:
  - name: ubuntu
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: sudo
    homedir: /home/ubuntu
    shell: /bin/bash
    ssh_authorized_keys:
      - $(cat ~/.ssh/id_ed25519.pub)

# Password setup
password: ubuntu
chpasswd:
  expire: false
ssh_pwauth: true

# Package management
package_update: true
package_upgrade: true
packages:
  - curl
  - apt-transport-https
  - ca-certificates
  - gnupg
  - containerd.io

write_files:
  - path: /etc/sysctl.d/k8s.conf
    content: |
      net.bridge.bridge-nf-call-ip6tables = 1
      net.bridge.bridge-nf-call-iptables = 1
      net.ipv4.ip_forward = 1
  - path: /etc/modules-load.d/k8s.conf
    content: |
      overlay
      br_netfilter
  - path: /etc/containerd/certs.d/docker.io/hosts.toml
    content: |
      server = "https://docker.io"
      [host."https://docker.m.daocloud.io"]
      capabilities = ["pull", "resolve"]
      [host."https://dockerproxy.net"]
      capabilities = ["pull", "resolve"]
  - path: /etc/containerd/certs.d/registry.k8s.io/hosts.toml
    content: |
      server = "https://registry.k8s.io"
      [host."https://k8s.m.daocloud.io"]
      capabilities = ["pull", "resolve"]
      [host."https://k8s.dockerproxy.net"]
      capabilities = ["pull", "resolve"]
  - path: /etc/containerd/certs.d/gcr.io/hosts.toml
    content: |
      server = "https://gcr.io"
      [host."https://gcr.m.daocloud.io"]
      capabilities = ["pull", "resolve"]
      [host."https://gcr.dockerproxy.net"]
      capabilities = ["pull", "resolve"]
  - path: /etc/containerd/certs.d/ghcr.io/hosts.toml
    content: |
      server = "https://ghcr.io"
      [host."https://ghcr.m.daocloud.io"]
      capabilities = ["pull", "resolve"]
      [host."https://ghcr.dockerproxy.net"]
      capabilities = ["pull", "resolve"]
  - path: /etc/containerd/config.toml
    content: |
      version = 2
      [plugins]
      [plugins."io.containerd.grpc.v1.cri"]
      [plugins."io.containerd.grpc.v1.cri".containerd]
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
      runtime_type = "io.containerd.runc.v2"
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
      SystemdCgroup = true
      [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = "/etc/containerd/certs.d"

# Commands to run at the end of the cloud-init process
runcmd:
  - curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
  - sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
  - sudo apt-get update
  - sudo apt-get install -y kubelet kubeadm kubectl kubernetes-cni
  - sudo apt-mark hold kubelet kubeadm kubectl
  - sudo systemctl enable --now containerd
  - sudo systemctl enable --now kubelet
  - sudo kubeadm config images pull --image-repository=registry.aliyuncs.com/google_containers

# Configure apt sources
apt:
  primary:
    - arches: [default]
      uri: https://mirrors.aliyun.com/ubuntu/
      search:
        - https://repo.huaweicloud.com/ubuntu/
        - https://mirrors.cloud.tencent.com/ubuntu/
        - https://mirrors.cernet.edu.cn/ubuntu/
        - https://archive.ubuntu.com
  sources:
    docker.list:
      source: deb [arch=amd64] https://mirrors.cernet.edu.cn/docker-ce/linux/ubuntu jammy stable
      keyid: 0EBFCD88
    kubernetes.list:
      source: deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /
      keyid: 0D811D58

power_state:
  mode: reboot

EOF
```

### 3.2 Deploy Virtual Machines

```bash
cp ${X_BASE_IMG} ${X_IMG_DIR}/k8s-cp-01.img
cp ${X_BASE_IMG} ${X_IMG_DIR}/k8s-worker-01.img
cp ${X_BASE_IMG} ${X_IMG_DIR}/k8s-worker-02.img

qemu-img resize ${X_IMG_DIR}/k8s-cp-01.img 20G
qemu-img resize ${X_IMG_DIR}/k8s-worker-01.img 20G
qemu-img resize ${X_IMG_DIR}/k8s-worker-02.img 20G

sed -i "s/^hostname: .*/hostname: k8s-cp-01/g" ${X_CFG_DIR}/cloud-init.yml
sed -i "s/^fqdn: .*/fqdn: k8s-cp-01.example.com/g" ${X_CFG_DIR}/cloud-init.yml
cloud-localds ${X_IMG_DIR}/k8s-cp-01.img.seed ${X_CFG_DIR}/cloud-init.yml

sed -i "s/^hostname: .*/hostname: k8s-worker-01/g" ${X_CFG_DIR}/cloud-init.yml
sed -i "s/^fqdn: .*/fqdn: k8s-worker-01.example.com/g" ${X_CFG_DIR}/cloud-init.yml
cloud-localds ${X_IMG_DIR}/k8s-worker-01.img.seed ${X_CFG_DIR}/cloud-init.yml

sed -i "s/^hostname: .*/hostname: k8s-worker-02/g" ${X_CFG_DIR}/cloud-init.yml
sed -i "s/^fqdn: .*/fqdn: k8s-worker-02.example.com/g" ${X_CFG_DIR}/cloud-init.yml
cloud-localds ${X_IMG_DIR}/k8s-worker-02.img.seed ${X_CFG_DIR}/cloud-init.yml

# Control Plane Node
sudo virt-install \
    --name k8s-cp-01 \
    --memory 2048 \
    --vcpus 2 \
    --disk path=${X_IMG_DIR}/k8s-cp-01.img,size=20 \
    --disk path=${X_IMG_DIR}/k8s-cp-01.img.seed,device=cdrom \
    --network network=${X_NET} \
    --os-variant ${X_OS_VARIANT} \
    --import \
    --graphics none \
    --noautoconsole

# Worker Node 01
sudo virt-install \
    --name k8s-worker-01 \
    --memory 2048 \
    --vcpus 2 \
    --disk path=${X_IMG_DIR}/k8s-worker-01.img,size=20 \
    --disk path=${X_IMG_DIR}/k8s-worker-01.img.seed,device=cdrom \
    --network network=${X_NET} \
    --os-variant ${X_OS_VARIANT} \
    --import \
    --graphics none \
    --noautoconsole

# Worker Node 02
sudo virt-install \
    --name k8s-worker-02 \
    --memory 2048 \
    --vcpus 2 \
    --disk path=${X_IMG_DIR}/k8s-worker-02.img,size=20 \
    --disk path=${X_IMG_DIR}/k8s-worker-02.img.seed,device=cdrom \
    --network network=${X_NET} \
    --os-variant ${X_OS_VARIANT} \
    --import \
    --graphics none \
    --noautoconsole
```

### 3.3 Wait for VM Boot

```bash
K8S_CP_IP=$(sudo virsh domifaddr "k8s-cp-01" | awk '/ipv4/ {print $4}' | cut -d'/' -f1)
K8S_WORKER_01_IP=$(sudo virsh domifaddr "k8s-worker-01" | awk '/ipv4/ {print $4}' | cut -d'/' -f1)
K8S_WORKER_02_IP=$(sudo virsh domifaddr "k8s-worker-02" | awk '/ipv4/ {print $4}' | cut -d'/' -f1)

echo "K8S_CP_IP: ${K8S_CP_IP}"
echo "K8S_WORKER_01_IP: ${K8S_WORKER_01_IP}"
echo "K8S_WORKER_02_IP: ${K8S_WORKER_02_IP}"

# Update /etc/hosts
sudo sed -i "/k8s-cp-01/d" /etc/hosts
sudo sed -i "/k8s-worker-01/d" /etc/hosts
sudo sed -i "/k8s-worker-02/d" /etc/hosts
echo "${K8S_CP_IP} k8s-cp-01" | sudo tee -a /etc/hosts >/dev/null
echo "${K8S_WORKER_01_IP} k8s-worker-01" | sudo tee -a /etc/hosts >/dev/null
echo "${K8S_WORKER_02_IP} k8s-worker-02" | sudo tee -a /etc/hosts >/dev/null

# Clean SSH known hosts
ssh-keygen -R "k8s-cp-01" >/dev/null 2>&1
ssh-keygen -R "k8s-worker-01" >/dev/null 2>&1
ssh-keygen -R "k8s-worker-02" >/dev/null 2>&1
ssh-keygen -R "${K8S_CP_IP}" >/dev/null 2>&1
ssh-keygen -R "${K8S_WORKER_01_IP}" >/dev/null 2>&1
ssh-keygen -R "${K8S_WORKER_02_IP}" >/dev/null 2>&1

# Retry the following command until cloud-init completes, maybe need to wait for 10 minutes
ssh ubuntu@k8s-cp-01 "test -f /var/lib/cloud/instance/boot-finished && echo 'cloud-init completed'"
ssh ubuntu@k8s-worker-01 "test -f /var/lib/cloud/instance/boot-finished && echo 'cloud-init completed'"
ssh ubuntu@k8s-worker-02 "test -f /var/lib/cloud/instance/boot-finished && echo 'cloud-init completed'"

# Continue to check some services
ssh ubuntu@k8s-cp-01 "sudo systemctl is-active containerd" # it will output "active" if containerd is running
ssh ubuntu@k8s-worker-01 "sudo systemctl is-active containerd"
ssh ubuntu@k8s-worker-02 "sudo systemctl is-active containerd"
```

## 4. Kubernetes Cluster Initialization

### 4.1 Control Plane Initialization

NOTES:
EOF with single quote to avoid variable expansion, i.e. it will keep the raw string
EOF with no single quote to expand variables

```bash
# Login to control plane node, initialize it
ssh ubuntu@k8s-cp-01 << EOF
sudo kubeadm init \
    --image-repository=registry.aliyuncs.com/google_containers \
    --kubernetes-version=v1.32.1 \
    --apiserver-advertise-address=${K8S_CP_IP} \
    --apiserver-bind-port=6443 \
    --pod-network-cidr=10.244.0.0/16 \
    --service-cidr=169.169.0.0/16 \
    --token=abcdef.0123456789abcdef \
    --token-ttl=0"
EOF

# Configure kubectl
ssh ubuntu@k8s-cp-01 << 'EOF' >/dev/null 2>&1
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
EOF

# Deploy Flannel network plugin
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

### 4.2 Add Worker Nodes

1. Join worker nodes:

    ```bash
    ssh ubuntu@k8s-worker-01 << EOF
    sudo $(ssh ubuntu@k8s-cp-01 kubeadm token create --print-join-command)
    EOF

    ssh ubuntu@k8s-worker-02 << EOF
    sudo $(ssh ubuntu@k8s-cp-01 kubeadm token create --print-join-command)
    EOF
    ```

2. (Optional) If kubectl access is required on worker nodes:

    ```bash
    # Login to worker node
    mkdir -p $HOME/.kube
    scp control-plane-node:/etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```

## 5. Cluster Verification

### 5.1 Basic Functionality Verification

```bash
# Check node status
kubectl get nodes -o wide

# Verify system components
kubectl get pods -n kube-system

# Verify CNI network plugin
kubectl get pods -n kube-flannel

# Deploy test application
kubectl create deployment nginx-test --image=nginx
kubectl expose deployment nginx-test --port=80 --type=NodePort
kubectl get pods,deployment,svc -o wide

# Access test application
curl -s $(kubectl get svc nginx-test -o jsonpath='{.spec.clusterIP}'):80

# Clean test application
kubectl delete service,deployment nginx-test
```

## 6. Maintenance Operations

### 6.1 Cluster Reset

To redeploy, execute on all nodes:

```bash
sudo kubeadm reset -f
sudo rm -rf $HOME/.kube
sudo rm -rf /etc/cni/net.d
```

### 6.2 Clean Virtual Environment

```bash
# Clean virtual machines
sudo virsh destroy k8s-cp-01
sudo virsh destroy k8s-worker-01
sudo virsh destroy k8s-worker-02
sudo virsh undefine k8s-cp-01 --remove-all-storage
sudo virsh undefine k8s-worker-01 --remove-all-storage
sudo virsh undefine k8s-worker-02 --remove-all-storage

# Clean network
sudo virsh net-destroy ${X_NET}
sudo virsh net-undefine ${X_NET}
```
