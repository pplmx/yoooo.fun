---
categories:
    - infrastructure
date: 2025-01-25T14:30:00Z
description: End-to-end automated Kubernetes cluster deployment solution using KVM virtualization and infrastructure-as-code practices.
keywords: kubernetes automation, kvm deployments, infrastructure-as-code, cluster orchestration, cloud-init, virsh
tags:
    - kubernetes
    - kvm
    - containerd
    - cloud-init
    - automation
    - virsh
title: Automated Kubernetes Deployments on KVM
---



# Automated Kubernetes Deployments on KVM

---

## 🚀 Core Features

The script provides the following core functionalities:

- Automatically creates `3` KVM VMs (`1` control plane + `2` worker nodes)
- Rapid deployment using Ubuntu `24.04` cloud image
- Auto-configures container runtime (containerd)
- Deploys Kubernetes `v1.32.1`
- Supports `Flannel` CNI plugin
- Provides complete cleanup functionality

---

## 🖥️ Complete Script Code

```bash
#!/bin/bash

# === Basic Configuration ===
VM_NAMES=("k8s-cp-01" "k8s-worker-01" "k8s-worker-02")
DISK_SIZE=20
RAM=4096
VCPUS=2
declare -A UBUNTU_VERSIONS=(
    ["ubuntu20.04"]="focal"
    ["ubuntu22.04"]="jammy"
    ["ubuntu24.04"]="noble"
)
OS_VARIANT="ubuntu24.04"
UBUNTU_NICKNAME="${UBUNTU_VERSIONS[${OS_VARIANT}]}"
IMG_URL="https://cloud-images.ubuntu.com/${UBUNTU_NICKNAME}/current/${UBUNTU_NICKNAME}-server-cloudimg-amd64.img"
K_DIR="/opt/k8s"
K_DOWNLOAD_DIR="${K_DIR}/download"
K_CONFIG_DIR="${K_DIR}/configs"
K_IMAGE_DIR="${K_DIR}/images"
CLOUD_IMAGE="${K_DOWNLOAD_DIR}/${UBUNTU_NICKNAME}-server-cloudimg-amd64.img"
NETWORK_NAME="k8s-net"
K8S_VERSION="v1.32.1"

# === Logging Function ===
log() {
    local GREEN='\033[0;32m'
    local YELLOW='\033[1;33m'
    local RED='\033[0;31m'
    local BLUE='\033[0;34m'
    local NC='\033[0m' # No Color
    local level=$1
    shift
    case ${level} in
        INFO)  echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')] [INFO]${NC} $*" ;;
        WARN)  echo -e "${YELLOW}[$(date +'%Y-%m-%d %H:%M:%S')] [WARN]${NC} $*" ;;
        ERROR) echo -e "${RED}[$(date +'%Y-%m-%d %H:%M:%S')] [ERROR]${NC} $*" ;;
        STEP)  echo -e "${BLUE}[$(date +'%Y-%m-%d %H:%M:%S')] [STEP]${NC} $*" ;;
        *)     echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')] [INFO]${NC} $*" ;;
    esac
}

# === Check Dependencies ===
check_deps() {
    log STEP "Checking system dependencies..."
    local deps=("wget" "virsh" "virt-install" "cloud-localds")
    for dep in "${deps[@]}"; do
        if ! command -v $dep &>/dev/null; then
            log INFO "Installing required packages..."
            sudo apt update && sudo apt install -y \
                wget qemu-system-x86 libvirt-daemon-system virtinst cloud-image-utils
            break
        fi
    done
}

# === Prepare Environment ===
prepare_env() {
    log STEP "Preparing environment and downloading resources..."
    sudo mkdir -p ${K_DIR} ${K_DOWNLOAD_DIR} ${K_CONFIG_DIR} ${K_IMAGE_DIR}
    sudo chown -R $USER:$USER ${K_DIR}

    # Download Ubuntu cloud image
    if [[ ! -f "${CLOUD_IMAGE}" ]]; then
        log INFO "Downloading Ubuntu cloud image..."
        wget -O "${CLOUD_IMAGE}" "${IMG_URL}"
    fi

    # Create virtual network
    if ! sudo virsh net-info "${NETWORK_NAME}" &>/dev/null; then
        log INFO "Creating virtual network: ${NETWORK_NAME}..."
        cat <<EOF > ${K_CONFIG_DIR}/${NETWORK_NAME}.xml
<network>
  <name>${NETWORK_NAME}</name>
  <forward mode="nat"/>
  <bridge name="virbr-k8s" stp="on" delay="0"/>
  <ip address="192.168.122.1" netmask="255.255.255.0">
    <dhcp>
      <range start="192.168.122.2" end="192.168.122.254"/>
    </dhcp>
  </ip>
</network>
EOF
        sudo virsh net-define ${K_CONFIG_DIR}/${NETWORK_NAME}.xml
        sudo virsh net-start ${NETWORK_NAME}
        sudo virsh net-autostart ${NETWORK_NAME}
    fi

    # Generate SSH key pair
    if [[ ! -f ~/.ssh/id_ed25519 ]]; then
        log INFO "Generating SSH key pair..."
        ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""
    fi
}

# === Generate Cloud-Init Configuration ===
gen_cloud_init() {
    local vm_name=$1
    log INFO "Generating cloud-init configuration for ${vm_name}..."
    cat <<EOF > ${K_CONFIG_DIR}/cloud-init.yaml
#cloud-config

# Basic system configuration
hostname: ${vm_name}
fqdn: ${vm_name}.example.com

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
      source: deb [arch=amd64] https://mirrors.cernet.edu.cn/docker-ce/linux/ubuntu ${UBUNTU_NICKNAME} stable
      keyid: 0EBFCD88
    kubernetes.list:
      source: deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /
      keyid: 0D811D58

power_state:
  mode: reboot

EOF
}

# === Create Virtual Machine ===
create_vm() {
    local vm_name=$1
    log STEP "Creating virtual machine: ${vm_name}..."

    local disk_path="${K_IMAGE_DIR}/${vm_name}.img"
    log INFO "Preparing disk image for ${vm_name}..."
    cp "${CLOUD_IMAGE}" "${disk_path}"
    qemu-img resize "${disk_path}" ${DISK_SIZE}G

    gen_cloud_init ${vm_name}
    cloud-localds "${disk_path}.seed" ${K_CONFIG_DIR}/cloud-init.yaml

    log INFO "Deploying virtual machine ${vm_name}..."
    sudo virt-install \
        --name "${vm_name}" \
        --ram "${RAM}" \
        --vcpus "${VCPUS}" \
        --disk path="${disk_path},size=${DISK_SIZE}" \
        --disk path="${disk_path}.seed",device=cdrom \
        --os-variant "${OS_VARIANT}" \
        --network network="${NETWORK_NAME}" \
        --graphics none \
        --import \
        --noautoconsole

    sudo virsh autostart "${vm_name}"
}

# === Check VM Status ===
check_vm_status() {
    local vm_name=$1
    local sleep_per_trial=10
    local max_tries=120
    local vm_ip=""
    local ssh_ready="false"
    local cloud_init_ready="false"

    log STEP "Waiting for VM ${vm_name} initialization..."

    for i in $(seq 1 $max_tries); do
        # Get VM IP
        vm_ip=$(sudo virsh domifaddr "$vm_name" | awk '/ipv4/ {print $4}' | cut -d'/' -f1)

        if [[ -n $vm_ip ]]; then
            # Check SSH readiness
            if [[ $ssh_ready == "false" ]]; then
                # Update /etc/hosts
                sudo sed -i "/$vm_name/d" /etc/hosts
                echo "$vm_ip $vm_name" | sudo tee -a /etc/hosts >/dev/null

                # Clean SSH known hosts
                ssh-keygen -R "$vm_name" >/dev/null 2>&1
                ssh-keygen -R "$vm_ip" >/dev/null 2>&1

                if ssh -o StrictHostKeyChecking=no -o ConnectTimeout=2 "ubuntu@$vm_name" "exit" >/dev/null 2>&1; then
                    log INFO "VM ${vm_name} SSH service is ready (IP: ${vm_ip})"
                    ssh_ready="true"
                fi
            fi

            # Check cloud-init status
            if [[ $ssh_ready == "true" && $cloud_init_ready == "false" ]]; then
                if ssh -o StrictHostKeyChecking=no -o ConnectTimeout=2 "ubuntu@$vm_name" \
                   "test -f /var/lib/cloud/instance/boot-finished && echo 'cloud-init completed'" >/dev/null 2>&1; then
                    log INFO "Cloud-init configuration completed for ${vm_name}"
                    cloud_init_ready="true"
                fi
            fi

            # Check containerd service
            if [[ $ssh_ready == "true" && $cloud_init_ready == "true" ]]; then
                if ssh "ubuntu@$vm_name" \
                    "systemctl is-active containerd" >/dev/null 2>&1; then
                    log INFO "Containerd service is running on ${vm_name}"
                    return 0
                fi
            fi
        fi

        if [[ $((i % 12)) -eq 0 ]]; then
            log INFO "Still waiting for ${vm_name} initialization... (${i}/${max_tries})"
        fi
        sleep $sleep_per_trial
    done

    if [[ $ssh_ready == "false" ]]; then
        log ERROR "SSH service not ready for ${vm_name}, timeout exceeded"
    fi

    if [[ $cloud_init_ready == "false" ]]; then
        log ERROR "Cloud-init configuration incomplete for ${vm_name}, timeout exceeded"
    fi

    return 1
}

# === Initialize Control Plane Node ===
init_control_plane() {
    log STEP "Initializing Kubernetes control plane node..."
    local cp_ip=$(sudo virsh domifaddr "k8s-cp-01" | awk '/ipv4/ {print $4}' | cut -d'/' -f1)

    log INFO "Running kubeadm init on control plane node..."
    ssh ubuntu@k8s-cp-01 << EOF || { log ERROR "Failed to configure kubectl"; return 1; }
sudo kubeadm init \
    --image-repository=registry.aliyuncs.com/google_containers \
    --kubernetes-version=${K8S_VERSION} \
    --apiserver-advertise-address=${cp_ip} \
    --apiserver-bind-port=6443 \
    --pod-network-cidr=10.244.0.0/16 \
    --service-cidr=169.169.0.0/16 \
    --token=abcdef.0123456789abcdef \
    --token-ttl=0
EOF

    # Configure kubectl
    log INFO "Configuring kubectl for ubuntu user..."
    ssh ubuntu@k8s-cp-01 << 'EOF'
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
EOF

    # Install CNI plugin
    log INFO "Installing CNI plugin (Flannel)..."
    ssh ubuntu@k8s-cp-01 "kubectl apply -f https://github.com/flannel-io/flannel/blob/master/Documentation/kube-flannel.yml"
}

# === Join Worker Nodes ===
join_workers() {
    log STEP "Adding worker nodes to the cluster..."
    local join_cmd=$(ssh ubuntu@k8s-cp-01 "sudo kubeadm token create --print-join-command")

    for vm in "k8s-worker-01" "k8s-worker-02"; do
        log INFO "Joining worker node: ${vm}"
        ssh ubuntu@${vm} "sudo ${join_cmd}" || {
            log ERROR "Failed to join ${vm} to the cluster"
            continue
        }
    done
}

# === Cleanup Function ===
cleanup() {
    log STEP "Cleaning up environment..."
    for vm in "${VM_NAMES[@]}"; do
        log INFO "Removing virtual machine: ${vm}"
        sudo virsh destroy ${vm} 2>/dev/null || true
        sudo virsh undefine ${vm} --remove-all-storage 2>/dev/null || true
    done

    log INFO "Removing virtual network: ${NETWORK_NAME}"
    sudo virsh net-destroy ${NETWORK_NAME} 2>/dev/null || true
    sudo virsh net-undefine ${NETWORK_NAME} 2>/dev/null || true

    log INFO "Cleaning up temporary files..."
    sudo rm -rf ${K_CONFIG_DIR}/* ${K_IMAGE_DIR}/*
}

main() {
    local CLEAN=0
    local INIT_ONLY=0
    local JOIN_ONLY=0

    while getopts "cij" opt; do
        case ${opt} in
            c) CLEAN=1 ;;
            i) INIT_ONLY=1 ;;
            j) JOIN_ONLY=1 ;;
            *) echo "Usage: $0 [-c] [-i] [-j]" && exit 1 ;;
        esac
    done

    [[ ${CLEAN} -eq 1 ]] && { cleanup; exit 0; }
    [[ ${INIT_ONLY} -eq 1 ]] && { init_control_plane; exit 0; }
    [[ ${JOIN_ONLY} -eq 1 ]] && { join_workers; exit 0; }

    check_deps
    prepare_env

    for vm in "${VM_NAMES[@]}"; do
        create_vm ${vm}
    done

    for vm in "${VM_NAMES[@]}"; do
        check_vm_status ${vm}
    done

    log INFO "All virtual machines are ready!"
    log INFO "        Control Plane Node: k8s-cp-01; try: ssh ubuntu@k8s-cp-01"
    log INFO "        Worker Node 1: k8s-worker-01; try: ssh ubuntu@k8s-worker-01"
    log INFO "        Worker Node 2: k8s-worker-02; try: ssh ubuntu@k8s-worker-02"
    log INFO "Run the following command to initialize the control plane node:"
    log INFO "        $0 -i"
    log INFO "If the flannel CNI plugin is not installed, run the following command:"
    log INFO "        Login: ssh ubuntu@k8s-cp-01"
    log INFO "        cd /var/tmp; curl -O https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml"
    log INFO "        kubectl apply -f /var/tmp/kube-flannel.yml"
    log INFO "Run the following command to join worker nodes to the cluster:"
    log INFO "        $0 -j"
}

main "$@"

```

---

## 🛠️ Usage Instructions

```bash
# Deploy KVMs
sudo ./deploy-k8s.sh

# Initialize control plane only
sudo ./deploy-k8s.sh -i

# Join worker nodes only
sudo ./deploy-k8s.sh -j

# Cleanup environment
sudo ./deploy-k8s.sh -c
```

---

## 💡 Best Practice Recommendations

1. Adjust `RAM` and `VCPUS` parameters based on hardware
2. Use dedicated disk partition for images in production
3. Modify `NETWORK_NAME` for multi-cluster isolation
4. Regularly clean expired images: `sudo rm -rf /opt/k8s/download/*`
5. Monitoring suggestion: Add `Prometheus` node exporter
