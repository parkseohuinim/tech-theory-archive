# Kubernetes PostgreSQL 가이드

## 목차

- [1. 설치(Helm)](#1-설치(Helm))
  - [1. 사전 준비(워커 노드)](#1-사전-준비(워커-노드))
  - [2. 로컬 스토리지 클래스 생성(마스터 노드)](#2-로컬-스토리지-클래스-생성(마스터-노드))
  - [3. PersistentVolume 생성(마스터 노드)](#3-PersistentVolume-생성(마스터-노드))
  - [4. Helm으로 PostgreSQL 설치(마스터 노드)](#4-Helm으로-PostgreSQL-설치(마스터-노드))
  - [5. 설치 확인(마스터 노드)](#5-설치-확인(마스터-노드))
  - [6. 비밀번호 확인 및 PostgreSQL 접속](#6-비밀번호-확인-및-PostgreSQL-접속)

## 1. 설치(Helm)

본 작업은 **마스터 노드**에서 **워커 노드**에 설치하는 작업

---

### 1. 사전 준비(워커 노드)

```bash
# 데이터 디렉토리 생성
sudo mkdir -p /mnt/postgres-data

sudo chmod 777 /mnt/postgres-data
```

### 2. 로컬 스토리지 클래스 생성(마스터 노드)

```bash
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
EOF
```

### 3. PersistentVolume 생성(마스터 노드)

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /mnt/postgres-data
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: node-role.kubernetes.io/worker-01
          operator: Exists
EOF
```

### 4. Helm으로 PostgreSQL 설치(마스터 노드)

```bash
helm install postgresql bitnami/postgresql \
  --set primary.nodeSelector."node-role\.kubernetes\.io/worker-01"=worker-01 \
  --set persistence.enabled=true \
  --set persistence.size=10Gi \
  --set auth.username=myuser \
  --set auth.password=mypassword \
  --set auth.database=mydb \
  --set auth.postgresPassword=rhkvqkq1990! \
  --set persistence.storageClass=local-storage
```

### 5. 설치 확인(마스터 노드)

```bash
kubectl get pods -l app.kubernetes.io/name=postgresql -o wide
kubectl get pvc -l app.kubernetes.io/name=postgresql
kubectl get pv
```

### 6. 비밀번호 확인 및 PostgreSQL 접속

```bash
# postgres 관리자 비밀번호 확인
echo $(kubectl get secret --namespace default postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

# PostgreSQL에 postgres 관리자로 접속
kubectl run postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:17.4.0-debian-12-r11 --env="PGPASSWORD=비밀번호입력" \
  --command -- psql --host postgresql -U postgres -d postgres -p 5432
```

---
