# Kubernetes 가이드

## 목차

- [1. kubernetes 설치 가이드(Amazon Linux 2023용)](#1-kubernetes-설치-가이드(Amazon-Linux-2023용))
  - [1. 스왑 비활성화(kubernetes 요구사항)](#1-스왑-비활성화(kubernetes-요구사항))
  - [2. 커널 모듈 활성화](#2-커널-모듈-활성화)
  - [3. 네트워크 설정](#3-네트워크-설정)
  - [4. 컨테이너 런타임 설치](#4-컨테이너-런타임-설치)
  - [5. 쿠버네티스 구성 요소 설치(바이너리 직접 설치)](#5-쿠버네티스-구성-요소-설치(바이너리-직접-설치))
  - [6. SELinux 설정](#6-SELinux-설정)
  - [7. 필요한 네트워크 도구 설치](#7-필요한-네트워크-도구-설치)
- [2. 마스터 노드 설정](#2-마스터-노드-설정)
  - [1. 초기화](#1-초기화)
  - [2. kubectl 설정](#2-kubectl-설정)
  - [3. 네트워크 플러그인 설치](#3-네트워크-플러그인-설치)
  - [4. 마스터 노드에서 Pod 실행 허용](#4-마스터-노드에서-Pod-실행-허용)
  - [5. 클러스터 상태 확인](#5-클러스터-상태-확인)

- [3. 워커 노드 설정](#3-워커-노드-설정)
  - [1. 클러스터에 조인](#1-클러스터에-조인)
  - [2. 조인 확인](#2-조인-확인)


## 1. kubernetes 설치 가이드(Amazon Linux 2023용)

### 1. 스왑 비활성화(kubernetes 요구사항)

- kubernetes는 스왑 메모리를 지원하지 않기 때문에, 스왑을 사용하면 컨테이너의 성능 예측이 어려워지고, 스케줄링 결정과 리소스 관리가 복잡해짐
- kubernetes는 pod에 할당된 메모리를 정확히 추적하고 관리해야하는데, 스왑이 활성화되면 이 과정이 방해됨

```bash
sudo swapoff -a

sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

---

### 2. 커널 모듈 활성화

- `overlay`: 컨테이너 파일 시스템에 필요한 모듈로, containerd와 같은 컨테이너 런타임 효율적인 레이어드 파일 시스템을 사용하도록 함
- `br_netfilter`: 브리지 네트워크 필터링을 활성화, 쿠버네티스 네트워킹과 포드간 통신에 필요

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay

sudo modprobe br_netfilter
```

---

### 3. 네트워크 설정

- `net.bridge.bridge-nf-call-iptables = 1`: 브리지로 연결된 네트워크 패킷이 iptables 규칙을 통과하도록 하며, kubernetes 네트워크 정책과 서비스가 제대로 작동하려면 필수
- `net.ipv4.ip_forward = 1`: IP 패킷 포워딩을 활성화하며, 컨테이너 간 통신 및 클러스터 네트워킹에 필요

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

---

### 4. 컨테이너 런타임 설치

- `yum-utils`: 저장소 관리 도구 및 유틸리티를 제공
- `device-mapper-persistent-data` & `lvm2`: 컨테이너 런타임이 스토리지를 관리하는데 필요한 논리적 볼륨 관리 도구

```bash
sudo dnf install -y yum-utils device-mapper-persistent-data lvm2
```

- Amazon Linux 2023에서는 기본 저장소에서 containerd 설치 (Docker 저장소 대신)

```bash
sudo dnf install -y containerd
```

- 기본 설정 파일을 생성
- `SystemdCgroup` = true: 리눅스의 systemd cgroup 드라이버를 사용하도록 설정, 이는 kubernetes 컨테이너 리소스를 효과적으로 관리하는데 중요

```bash
sudo mkdir -p /etc/containerd

containerd config default | sudo tee /etc/containerd/config.toml

sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```

- containerd 서비스를 시작하고, 부팅 시 자동으로 시작되도록 설정

```bash
sudo systemctl restart containerd

sudo systemctl enable containerd
```

---

### 5. 쿠버네티스 구성 요소 설치(바이너리 직접 설치)

- Amazon Linux 2023에서는 쿠버네티스 저장소가 호환되지 않으므로 바이너리 직접 설치 방식 사용

```bash
K8S_VERSION=v1.29.4

# 각 바이너리 다운로드
curl -LO "https://dl.k8s.io/release/${K8S_VERSION}/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/${K8S_VERSION}/bin/linux/amd64/kubeadm"
curl -LO "https://dl.k8s.io/release/${K8S_VERSION}/bin/linux/amd64/kubelet"

# 실행 권한 부여
chmod +x kubectl kubeadm kubelet

# 시스템 디렉토리로 이동
sudo mv kubectl kubeadm /usr/local/bin/
sudo mv kubelet /usr/bin/

# kubelet 서비스 파일 다운로드
sudo mkdir -p /etc/systemd/system/kubelet.service.d

curl -sSL "https://raw.githubusercontent.com/kubernetes/release/v0.16.2/cmd/kubepkg/templates/latest/deb/kubelet/lib/systemd/system/kubelet.service" | sudo tee /etc/systemd/system/kubelet.service

# kubelet 서비스 설정 파일
curl -sSL "https://raw.githubusercontent.com/kubernetes/release/v0.16.2/cmd/kubepkg/templates/latest/deb/kubeadm/10-kubeadm.conf" | sudo tee /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

# crictl 설치 (쿠버네티스용 컨테이너 런타임 인터페이스 도구)
CRICTL_VERSION=v1.29.0
curl -L "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz" | sudo tar -C /usr/local/bin -xz

# kubelet 서비스 활성화
sudo systemctl enable --now kubelet
```

---

### 6. SELinux 설정

- `SELinux`를 permissive 모드로 설정하여 쿠버네티스 호환성 확보

```bash
sudo setenforce 0

sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

---

### 7. 필요한 네트워크 도구 설치

- 쿠버네티스가 필요로 하는 네트워크 유틸리티들 설치

```bash
sudo dnf install -y conntrack-tools iptables iproute-tc socat
```

---

## 2. 마스터 노드 설정

필수 조건: 설정 전 [1. kubernetes 설치 가이드(Amazon Linux 2023용)](#1-kubernetes-설치-가이드(Amazon-Linux 2023용)) 필수

---

### 1. 초기화

- 쿠버네티스 클러스터 초기화 (API 서버를 퍼블릭 IP로 노출)

```bash
PUBLIC_IP=$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)

# 경고 무시하고 클러스터 초기화
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --control-plane-endpoint=${PUBLIC_IP} --apiserver-advertise-address=$(hostname -i) --ignore-preflight-errors=FileExisting-ebtables,FileExisting-tc
```

---

### 2. kubectl 설정

- 일반 사용자가 `kubectl`을 사용할 수 있도록 설정

```bash
mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

### 3. 네트워크 플러그인 설치

- `Calico CNI` 플러그인 설치로 Pod 네트워킹 활성화

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

---

### 4. 마스터 노드에서 Pod 실행 허용

- 개발 환경에서는 마스터 노드에서도 워크로드 실행 허용

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

---

### 5. 클러스터 상태 확인

- 노드와 시스템 Pod 상태 확인

```bash
kubectl get nodes

kubectl get pods --all-namespaces
```

---

## 3. 워커 노드 설정

### 1. 클러스터에 조인

```bash
sudo kubeadm join [마스터_노드_IP]:6443 --token [토큰] \
    --discovery-token-ca-cert-hash sha256:[해시]
    
# 마스터 노드에서 실행
kubeadm token create --print-join-command
```

---

### 2. 조인 확인

```bash
# 마스터 노드에서 실행
kubectl get nodes
```

---
