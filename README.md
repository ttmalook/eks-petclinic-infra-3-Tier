
# 🏥 EKS 기반 Spring PetClinic 3-Tier 자동화 배포

## 🗺️ 프로젝트 오버뷰

이 저장소는 Spring 기반의 오픈소스 웹 애플리케이션인 **PetClinic**을 AWS EKS 기반 클러스터에 완전 자동화 방식으로 배포하기 위한 예제입니다.

GitHub Actions를 통한 CI/CD 파이프라인 자동화, Helm Chart 기반의 Kubernetes 애플리케이션 배포, AWS EFS를 통한 로그 저장소 연동, CloudFormation 기반의 네트워크(VPC) 구성까지 **모든 인프라 및 애플리케이션 레벨 구성을 하나의 리포지토리로 통합**하였습니다.

이 프로젝트는 다음과 같은 목적으로 구성하였습니다:

- **DevOps 실습** 또는 사내 DevOps 표준 프로세스 구축
- Kubernetes 및 EKS 학습 및 구조 이해
- 자동화된 클라우드 인프라 배포 예제
- Spring 기반 웹 애플리케이션의 마이크로서비스 전환 준비

---

## ✅ 프로젝트 개요

이 프로젝트는 Spring PetClinic 웹 애플리케이션을 AWS 클라우드 환경에 자동으로 설치하고 운영하기 위한 인프라 자동화 예시입니다.

### 주요 구성

- **EKS (Elastic Kubernetes Service)**: 컨테이너 오케스트레이션 플랫폼으로, Kubernetes 클러스터를 손쉽게 운영 가능하게 해줍니다.
- **VPC (Virtual Private Cloud)**: 애플리케이션과 리소스를 위한 격리된 네트워크 환경입니다. 퍼블릭/프라이빗 서브넷으로 구성되어 외부와 내부 통신을 분리합니다.
- **EFS (Elastic File System)**: 여러 노드에서 동시에 접근 가능한 공유 파일 스토리지입니다. 주로 로그나 설정 데이터를 저장할 때 사용됩니다.
- **Helm Chart**: Kubernetes 앱을 템플릿 기반으로 선언적 구성으로 배포하는 도구입니다.
- **GitHub Actions**: 코드 변경 또는 수동 실행을 통해 CI/CD 파이프라인을 자동으로 수행하는 GitHub의 워크플로우 서비스입니다.

---

## 🧭 자동화 동작 흐름 요약

![EKS 기반 자동화 다이어그램]([https://files.chat.openai.com/file-HjgxSL6Xm1cebNfkcyL9c6](https://github.com/ttmalook/eks-petclinic-infra-3-Tier/blob/main/ChatGPT%20Image%202025%EB%85%84%205%EC%9B%94%2030%EC%9D%BC%20%EC%98%A4%ED%9B%84%2002_25_34.png?raw=true)

> 위 다이어그램은 GitHub Actions를 통해 실행되는 자동화된 배포 과정을 시각적으로 보여줍니다. 
> VPC → EKS → EFS → PetClinic 애플리케이션으로 이어지는 단계가 명확하게 구분되며, 각 리소스는 독립적이면서도 상호 연결되어 있습니다. 
> 모든 단계는 코드 기반으로 정의되어 GitHub에 커밋하거나 수동 실행을 통해 간편하게 배포할 수 있습니다.

1. **create-vpc.yaml**

   - CloudFormation을 사용하여 3-Tier VPC 구성
   - 퍼블릭 서브넷 (ALB 등 외부 접속), 프라이빗 서브넷 (EKS NodeGroup용), 라우팅 테이블, NAT Gateway 등 포함

2. **create-eks.yaml**

   - `eksctl` 기반으로 EKS 클러스터 및 관리형 노드 그룹(MNG) 생성
   - OIDC Provider, IAM Role for Service Account(IRSA) 설정도 자동화 가능

3. **deploy-efs.yaml**

   - AWS CLI 또는 kubectl 기반으로 EFS 파일시스템 생성
   - 모든 가용영역(AZ)에 Mount Target을 자동 생성하여 고가용성 구성
   - StorageClass, PersistentVolume, PersistentVolumeClaim 템플릿을 Helm에서 동적으로 처리

4. **deploy-helm-petclinic.yaml**

   - `charts/petclinic/` 경로의 Helm Chart를 이용하여 Web/WAS 배포
   - WAS는 EFS 볼륨을 마운트하고 Aurora MySQL과 연결
   - Web은 httpd 이미지 기반으로 ConfigMap을 통해 Apache 설정
   - Ingress를 통해 ALB 연동 가능

---

## 📁 디렉토리 구조 요약

```
eks-petclinic-infra/
├── eks-cluster.yaml              # eksctl로 EKS 클러스터 구성
├── vpc-3tier.yaml                # CloudFormation 기반 3-Tier VPC 구성
├── .github/workflows/           # GitHub Actions 자동화 파일
│   ├── create-vpc.yaml
│   ├── create-eks.yaml
│   ├── deploy-efs.yaml
│   └── deploy-helm-petclinic-commented.yaml
└── charts/petclinic/            # Helm Chart
    ├── Chart.yaml               # Helm 메타정보
    ├── values.yaml              # 사용자 정의 값
    ├── scripts/                 # 보안그룹 등 부가 스크립트
    └── templates/               # Kubernetes 리소스 템플릿
        ├── deployment-web.yaml      # Apache 웹서버 배포
        ├── deployment-was.yaml      # Spring WAS 배포
        ├── service-web.yaml         # Web Service (LoadBalancer)
        ├── service-was.yaml         # WAS 내부 서비스
        ├── ingress.yaml             # ALB Ingress 설정
        ├── configmap.yaml           # WAS/Web 설정파일
        ├── secrets.yaml             # Aurora MySQL 접속 정보
        ├── pv.yaml / pvc.yaml       # EFS용 볼륨 매핑
```

---

## 🔄 실행 순서 흐름도

```
[GitHub Push or Dispatch Event]
        ↓
1️⃣ create-vpc.yaml → VPC + Subnet + IGW + NATGW 생성
        ↓
2️⃣ create-eks.yaml → EKS 클러스터 및 노드그룹 구성
        ↓
3️⃣ deploy-efs.yaml → EFS 파일시스템 및 Mount Target 구성
        ↓
4️⃣ deploy-helm-petclinic.yaml → Helm Chart로 Web/WAS 배포
```

---

## 📌 사용 방법

1. 이 저장소를 Fork 또는 Clone 합니다.
2. GitHub Secrets에 다음 AWS 자격 증명을 등록합니다.
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`
3. GitHub Actions에서 각 워크플로우를 수동 실행하거나, 커밋을 푸시해 자동 실행합니다.

> Aurora DB는 미리 생성되어 있어야 하며, `values.yaml` 또는 `secrets.yaml`에 관련 정보가 설정되어 있어야 합니다.

---

## 🏁 요약

- AWS 인프라(VPC, EKS, EFS)를 GitHub Actions로 자동화 구성
- Helm을 활용해 Spring PetClinic 앱을 안정적으로 배포
- 실습 및 PoC에 적합한 자동화 

