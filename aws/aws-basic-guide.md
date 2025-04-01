# AWS 가이드

## 목차

- [1. EC2](#1-EC2)
  - [SSH로 EC2 인스턴스에 연결하기](#SSH로-EC2-인스턴스에-연결하기)
  - [명령어](#명령어)
  - [AWS 자격 증명 구성](#AWS-자격-증명-구성)


## 1. EC2

### SSH로 EC2 인스턴스에 연결하기

```bash
ssh -i seohui_key.pem ec2-user@your-ec2-public-ip
```

---

### 명령어 

```bash
# 시스템 업데이트
sudo dnf update -y

# AWS CLI가 설치되어 있는지 확인
aws --version

# 설치되어 있지 않다면
sudo dnf install -y aws-cli
```

---

### AWS 자격 증명 구성

```bash
aws configure
# 1. AWS Access Key ID 입력
# 2. AWS Scret Access Key 입력
# 3. 기본 리전 입력(서울 예: ap-northeast-2)
# 4. 기본 출력 형식 입력(예: json)
```

---