# AWS 스팟 인스턴스 Kubernetes 백업 및 복구 전략

## 목차

- [1. 스팟 인스턴스의 특성](#1-스팟-인스턴스의-특성)
- [2. 백업 전략: AMI 생성](#2-백업-전략-AMI-생성)
  - [AMI 생성 과정](#AMI-생성-과정)

- [3. 복구 전략: AMI에서 새 인스턴스 시작](#3-복구-전략-AMI에서-새-인스턴스-시작)
  - [AWS 콘솔에서 복구](#AWS-콘솔에서-복구)
  - [AWS CLI로 복구](#AWS-CLI로-복구)
  - [인스턴스 시작 후 추가 설정](#인스턴스-시작-후-추가-설정)
  - [정기적인 백업 자동화](#정기적인-백업-자동화)

- [4. 결론](#4-결론)

---

이 가이드는 AWS EC2 스팟 인스턴스에서 실행 중인 kubernetes 클러스터를 백업하고, 인스턴스 종료 시 복구하는 방법을 안내

---

## 1. 스팟 인스턴스의 특성

- 스팟 인스턴스는 온디맨드 대비 최대 90%까지 비용 절감 가능
- AWS에 용량이 부족할 경우 2분 알림 후 인스턴스가 종료될 수 있음
- stop(중지) 불가능: stop하려고 시도하면 terminate(종료)됨
- terminate 시 인스턴스가 완전히 삭제되므로 백업 전략이 필요

---

## 2. 백업 전략: AMI 생성

`AMI(Amazon Machine Image)`는 인스턴스의 전체 상태를 캡처하는 가장 안전하고 간편한 복구 방법

---

### AMI 생성 과정

1. **EC2 인스턴스에 SSH로 접속:**

```bash
ssh -i your-key.pem ec2-user@your-instance-ip
```

2. **IMDSv2 토큰을 요청하고 인스턴스 ID 가져오기:**

```bash
# 169.254.169.254는 AWS EC2 인스턴스 메타데이터 서비스의 IP 주소
# 이 주소는 모든 EC2 인스턴스 내부에서만 접근할 수 있는 특별한 주소로, 인스턴스 자체에 대한 정보를 제공
# EC2 인스턴스 내에서 이 주소로 HTTP 요청을 보내면 해당 인스턴스의 메타데이터(인스턴스 ID, 퍼블릭 IP, 보안 그룹, IAM 역할 등)를 얻을 수 있음 
# 이것은 AWS 인프라의 일부이며, 인스턴스가 자신에 대한 정보를 프로그래밍 방식으로 얻을 수 있음

# 토큰 요청
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

# 인스턴스 ID 가져오기
INSTANCE_ID=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id)

# 확인
echo $INSTANCE_ID
```

3. **AMI 생성:**

```bash
aws ec2 create-image \
  --instance-id "$INSTANCE_ID" \
  --name "k8s-master-$(date +%Y%m%d)" \
  --description "Kubernetes master node backup" \
  --no-reboot
```

4. **AMI 생성 상태 확인:**

```bash
aws ec2 describe-images --owners self --filters "Name=name,Values=k8s-master-*" --query 'Images[*].[ImageId,Name,CreationDate]' --output table
```

## 3. 복구 전략: AMI에서 새 인스턴스 시작

스팟 인스턴스가 종료된 후 `AMI`에서 새 인스턴스를 시작하여 복구할 수 있음

---

### AWS 콘솔에서 복구

1. **AWS 관리 콘솔에 로그인**
2. **EC2 서비스로 이동**
3. **왼쪽 메뉴에서 `AMI` 선택**
4. **이전에 생성한 "k8s-master-YYYYMMDD" `AMI` 선택**
5. **`인스턴스 시작` 버튼 클릭**
6. **`t3.medium` 등 원하는 인스턴스 유형 선택**
7. **`고급 세부 정보`에서 `스팟 인스턴스 요청` 옵션 선택 (스팟으로 실행하려는 경우)**
8. **`보안 그룹`, `키 페어` 등 설정 후 `인스턴스 시작`**

---

### AWS CLI로 복구

```bash
# AMI ID 확인
aws ec2 describe-images --owners self --filters "Name=name,Values=k8s-master-*" --query 'Images[*].[ImageId,Name,CreationDate]' --output table

# 새 인스턴스 시작
aws ec2 run-instances \
  --image-id ami-0123456789abcdef0 \ # 실제 AMI ID로 교체
  --instance-type t3.medium \
  --key-name your-key-name \ # 실제 키 페어 이름으로 교체
  --security-group-ids sg-0123456789abcdef0 \ # 실제 보안 그룹 ID로 교체
  --instance-market-options '{"MarketType":"spot"}' \ # 스팟으로 실행하려면 포함
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=k8s-master}]'
```

---

### 인스턴스 시작 후 추가 설정

새 인스턴스가 시작된 후, IP 주소가 변경되었기 때문에 kubernetes 설정을 업데이트해야 할 수 있음

```bash
# SSH로 새 인스턴스에 접속
ssh -i your-key.pem ec2-user@new-instance-ip

# 새 IP 주소 가져오기
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
PUBLIC_IP=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/public-ipv4)

# kube-apiserver의 IP 주소 업데이트 (필요한 경우)
sudo sed -i "s/--advertise-address=.*/--advertise-address=${PUBLIC_IP} \\\\/" /etc/kubernetes/manifests/kube-apiserver.yaml

# kubelet 재시작
sudo systemctl restart kubelet
```

---

### 정기적인 백업 자동화

`cron` 작업을 설정하여 정기적으로 `AMI`를 생성하는 스크립트:

```bash
# 아래의 스크립트를 /home/ec2-user/backup-ami.sh로 저장
#!/bin/bash

# 필요한 변수 설정
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
INSTANCE_ID=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id)
DATE=$(date +%Y%m%d)
LOG_FILE="/home/ec2-user/ami-backup-$DATE.log"

echo "$(date): 백업 스크립트 시작" > $LOG_FILE

# AMI 생성
echo "$(date): AMI 생성 중..." >> $LOG_FILE
AMI_RESULT=$(aws ec2 create-image \
  --instance-id "$INSTANCE_ID" \
  --name "k8s-master-${DATE}" \
  --description "Kubernetes master node backup" \
  --no-reboot)

# AMI ID 추출
AMI_ID=$(echo $AMI_RESULT | grep -o 'ami-[a-z0-9]*')
echo "$(date): 생성된 AMI ID: $AMI_ID" >> $LOG_FILE

# AMI에 태그 추가
aws ec2 create-tags \
  --resources "$AMI_ID" \
  --tags Key=AutoBackup,Value=true

# 7일 이상 지난 AMI 삭제
echo "$(date): 오래된 AMI 정리 시작" >> $LOG_FILE
# 7일 전 날짜 계산
OLD_DATE=$(date -d "7 days ago" +%Y%m%d)

# 소유한 모든 AMI 목록 가져오기
OLD_AMIS=$(aws ec2 describe-images \
  --owners self \
  --filters "Name=name,Values=k8s-master-*" "Name=tag:AutoBackup,Values=true" \
  --query "Images[?CreationDate<='${OLD_DATE}T00:00:00.000Z'].ImageId" \
  --output text)

# 오래된 AMI가 있으면 삭제
if [ ! -z "$OLD_AMIS" ]; then
  for OLD_AMI in $OLD_AMIS; do
    echo "$(date): 오래된 AMI 삭제 중: $OLD_AMI" >> $LOG_FILE
    
    # 관련 스냅샷 ID 찾기
    SNAPSHOTS=$(aws ec2 describe-images \
      --image-ids "$OLD_AMI" \
      --query 'Images[0].BlockDeviceMappings[*].Ebs.SnapshotId' \
      --output text)
    
    # AMI 등록 취소
    aws ec2 deregister-image --image-id "$OLD_AMI"
    echo "$(date): AMI 등록 취소 완료: $OLD_AMI" >> $LOG_FILE
    
    # 관련 스냅샷 삭제
    for SNAPSHOT in $SNAPSHOTS; do
      echo "$(date): 스냅샷 삭제 중: $SNAPSHOT" >> $LOG_FILE
      aws ec2 delete-snapshot --snapshot-id "$SNAPSHOT"
    done
  done
else
  echo "$(date): 삭제할 오래된 AMI가 없습니다" >> $LOG_FILE
fi

echo "$(date): 백업 스크립트 완료" >> $LOG_FILE
```

`cron` 작업 설정:

```bash
chmod +x /home/ec2-user/backup-ami.sh
(crontab -l 2>/dev/null; echo "0 1 * * * /home/ec2-user/backup-ami.sh") | crontab -
```

---

## 4. 결론

- 스팟 인스턴스의 비용 효율성을 활용하면서도 AMI 백업을 통해 갑작스러운 인스턴스 종료에도 빠르게 복구할 수 있음 
- 개발 환경이나 테스트 환경에서는 이 방법이 비용 대비 효율적인 전략으로, 중요한 프로덕션 환경에서는 온디맨드 인스턴스와 스팟 인스턴스를 혼합하여 사용하는 것을 고려