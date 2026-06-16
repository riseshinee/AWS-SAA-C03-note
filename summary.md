# AWS SAA-C03 시험 핵심 정리

## 📦 스토리지

### S3 (Simple Storage Service)

- **용도**: 객체 스토리지, 무제한 확장
- **스토리지 클래스**: Standard → IA → One Zone-IA → Intelligent-Tiering → Glacier → Glacier Deep Archive
- 버킷 정책 + ACL + IAM으로 접근 제어
- 정적 웹사이트 호스팅 가능
- 멀티파트 업로드 (파일 >100MB 권장)
- 버전 관리, 복제(CRR/SRR), 라이프사이클 정책
- Pre-Signed URL로 임시 접근 허용
- 암호화: SSE-S3, SSE-KMS, SSE-C, 클라이언트 암호화
- ⚠️ **최소 3개 AZ 중복** (One Zone-IA 제외)

#### S3 스토리지 클래스 상세 비교

| 클래스 | 특징 | 사용 사례 |
|--------|------|-----------|
| **S3 Standard** | 상시 접근, 고성능, 밀리초 단위 응답 | 웹 자산, 활성 데이터 |
| **S3 Intelligent-Tiering** | 액세스 패턴을 모를 때 사용, 자동 계층 이동 (모니터링 비용 발생, 전환 수수료 없음) | 패턴 불명확한 데이터 |
| **S3 Glacier Deep Archive** | 가장 저렴, 복구 시간 최대 12시간 | 규정 준수 및 아카이브 백업 |

---

### EBS (Elastic Block Store)

- **용도**: EC2 전용 블록 스토리지 (단일 AZ)
- 유형: gp2/gp3(범용), io1/io2(고성능), st1(처리량 HDD), sc1(저비용 HDD)
- gp3: 기본 3,000 IOPS / 최대 16,000 IOPS
- io2 Block Express: 최대 256,000 IOPS
- io1/io2: **Multi-Attach** 가능 (여러 EC2 연결)
- 스냅샷으로 다른 AZ/리전 이동 가능
- ⚠️ **단일 AZ에 묶임** → 이동 시 스냅샷 사용

---

### EFS (Elastic File System)

- **용도**: 여러 EC2가 동시 마운트하는 NFS 파일 시스템
- 자동 확장/축소 (용량 사전 프로비저닝 불필요)
- 성능 모드: General Purpose / Max I/O
- 스토리지 클래스: Standard / IA (Lifecycle 정책으로 자동 이동)
- ⚠️ **Linux only** (Windows 지원 X) / EBS보다 비용 높음
- Multi-AZ 내구성

---

### FSx

| 유형 | 특징 | 사용 대상 |
|------|------|-----------|
| **FSx for Windows** | SMB/NTFS, Active Directory 통합 | Windows EC2 |
| **FSx for Lustre** | HPC, ML, 고성능 컴퓨팅, S3 연동 | HPC/ML 워크로드 |
| **FSx for NetApp ONTAP** | 멀티 프로토콜 지원 | 멀티 환경 |

> ⚠️ EFS = Linux NFS / FSx for Windows = Windows SMB

---

### Storage Gateway

- **용도**: 온프레미스 ↔ AWS 스토리지 브리지

| 유형 | 설명 |
|------|------|
| **File Gateway** | S3에 NFS/SMB로 파일 저장 |
| **Volume Gateway** | iSCSI 블록 스토리지 (Cached/Stored 모드) |
| **Tape Gateway** | 가상 테이프 라이브러리 → Glacier |

---

## 💻 컴퓨팅

### EC2 (Elastic Compute Cloud)

#### 구매 옵션 비교

| 구매 옵션 | 비용 절감률 | 주요 특징 |
|-----------|-------------|-----------|
| **On-Demand** | 기준 가격 | 약정 없음, 초 단위 과금. 단기·예측 불가 워크로드 |
| **Reserved Instances** | 최대 72% | 1년/3년 약정. 안정적인 상시 워크로드 (예: DB) |
| **Savings Plans** | 최대 72% | RI와 동일 할인율, 인스턴스 패밀리/리전 변경 유연 |
| **Spot Instances** | 최대 90% | AWS 유휴 자원, 언제든 회수 가능 (2분 전 알림). 배치·HPC |
| **Dedicated Host** | 가장 비쌈 | 전용 물리 서버. BYOL 라이선스·규정 준수 목적 |

- **배치 그룹**: Cluster(저지연) / Spread(고가용성) / Partition(대규모 분산)
- **인스턴스 유형**: T(버스트), M(범용), C(컴퓨팅), R(메모리), I(스토리지), P(GPU)
- ⚠️ User Data로 부팅 시 스크립트 실행

---

### Lambda

- **용도**: 서버리스 함수, 이벤트 기반 실행
- 최대 실행 시간: **15분** / 메모리: 128MB ~ 10GB
- 동시 실행 한도: 기본 1,000 (Reserved Concurrency로 제한)
- **Provisioned Concurrency**: 콜드 스타트 방지
- 이벤트 소스: API Gateway, S3, DynamoDB, SQS, SNS 등
- VPC 내 배포 가능 (인터넷 접근 시 NAT Gateway 필요)
- ⚠️ **15분 초과** 작업 → Step Functions 또는 Fargate

---

### ECS / EKS / Fargate

| 서비스 | 설명 |
|--------|------|
| **ECS** | AWS 관리형 컨테이너 (EC2 또는 Fargate 실행) |
| **EKS** | 관리형 Kubernetes |
| **Fargate** | 서버리스 컨테이너 (인프라 관리 불필요) |

> ⚠️ EC2 모드 = 인프라 직접 관리 / Fargate = 서버리스

---

### Elastic Beanstalk

- **용도**: 코드만 올리면 EC2, LB, ASG, RDS 자동 프로비저닝
- 지원 언어: Java, .NET, PHP, Node.js, Python, Ruby, Go, Docker
- **배포 방식**: All at once / Rolling / Rolling with additional batch / Immutable / Blue/Green
- ⚠️ 인프라 제어 줄이고 빠른 배포 원할 때

---

### CloudFormation

- **용도**: IaC(Infrastructure as Code)로 인프라 코드화
- YAML/JSON 템플릿으로 AWS 리소스 정의
- 스택 단위 배포/삭제
- 드리프트 감지: 실제 인프라와 템플릿 차이 확인
- ⚠️ 인프라 재현성, 버전 관리에 핵심

---

## 🗄️ 데이터베이스

### RDS (Relational Database Service)

- **지원 DB**: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Aurora
- **Multi-AZ**: 자동 장애 조치 (동기 복제) → 고가용성
- **Read Replica**: 읽기 성능 확장 (비동기 복제, 최대 5개)
- **RDS Proxy**: Lambda와 RDS 연결 시 커넥션 풀링
- 자동 백업, 스냅샷 지원
- ⚠️ Multi-AZ = 고가용성 / Read Replica = 성능 향상

---

### Aurora

- **용도**: AWS 고성능 관계형 DB (MySQL/PostgreSQL 호환)
- MySQL 대비 최대 5배 성능
- 6개 복사본을 3개 AZ에 자동 저장
- **Aurora Serverless**: 자동 용량 조정
- **Aurora Global Database**: 여러 리전 복제 (RPO 1초)
- ⚠️ 고성능 + 고가용성 관계형 DB = Aurora 선택

---

### DynamoDB

- **용도**: 서버리스 NoSQL, 밀리초 응답
- 키-값 + 문서 모델, 자동 확장
- **DAX**: DynamoDB Accelerator (마이크로초 캐시)
- **DynamoDB Streams**: 변경 이벤트 캡처 → Lambda 트리거
- **Global Tables**: 멀티 리전 복제
- TTL로 아이템 자동 만료
- ⚠️ ACID 트랜잭션 지원 / 최대 아이템 크기 400KB

---

### ElastiCache

| 엔진 | 특징 |
|------|------|
| **Redis** | 복제, 영속성, 클러스터, Pub/Sub, 정렬 집합 |
| **Memcached** | 단순 캐시, 멀티스레드, 영속성 X |

> ⚠️ 세션 저장 / DB 부하 감소 / 리더보드 = Redis

---

### Redshift

- **용도**: 데이터 웨어하우스, OLAP 분석
- 컬럼 기반 스토리지, 대규모 분석 쿼리
- **Redshift Spectrum**: S3 데이터를 직접 쿼리
- ⚠️ OLTP = RDS/Aurora / OLAP/분석 = Redshift
- 단일 AZ (Multi-AZ 미지원, 스냅샷으로 복구)

---

### Amazon QuickSight

- **용도**: 서버리스 비즈니스 인텔리전스(BI) 및 데이터 시각화 서비스
- 다양한 데이터 소스(RDS, Redshift, S3, 온프레미스 등)와 연결하여 대시보드 생성
- **SPICE 엔진**: 메모리 내 계산 엔진으로 대규모 데이터 시각화 쿼리 고속 처리
- ⚠️ **시험 함정**: "대시보드 시각화", "비즈니스 보고서 생성" → QuickSight

---

## 🌐 네트워킹

### VPC (Virtual Private Cloud)

| 구분 | Public Subnet | Private Subnet (앱) | Private Subnet (DB) |
|------|---------------|----------------------|----------------------|
| 라우팅 | `0.0.0.0/0 → IGW` | `0.0.0.0/0 → NAT GW` | 로컬만 (인터넷 경로 없음) |
| 인터넷 접근 | 인바운드 + 아웃바운드 가능 | 아웃바운드만 (NAT 경유) | 완전 차단 |
| 배치 리소스 | ALB, Bastion Host, NAT GW | EC2 앱 서버, Lambda | RDS, Aurora, ElastiCache |

- **NACL**: 서브넷 레벨, Stateless (in/out 별도 규칙)
- **Security Group**: 인스턴스 레벨, Stateful
- **VPC Peering**: VPC 간 연결 (전이 X)
- **Transit Gateway**: 다수 VPC/온프레미스 중앙 허브
- **VPC Endpoint**: 인터넷 없이 AWS 서비스 접근

> ⚠️ **핵심 암기**: 서브넷이 퍼블릭인지 여부는 **라우팅 테이블의 0.0.0.0/0 목적지**로 결정
> - IGW → 퍼블릭
> - NAT GW → 프라이빗 (아웃바운드 가능)
> - 없음 → 완전 격리

---

### Security Group vs NACL

| 구분 | Security Group (SG) | NACL |
|------|---------------------|------|
| 적용 대상 | **인스턴스(ENI) 레벨** | **서브넷 레벨** |
| 상태 관리 | **Stateful** | **Stateless** |
| 규칙 종류 | 허용(Allow)만 가능 | 허용(Allow) + **거부(Deny) 가능** |
| 규칙 평가 | 모든 규칙 평가 후 허용 | **번호 순 평가, 첫 매칭에서 종료** |
| 기본 동작 | 인바운드 전체 거부 / 아웃바운드 전체 허용 | 기본 전체 허용(100번 규칙) |

> ⚠️ **가장 많이 틀리는 포인트**
> - "특정 IP 차단" → **NACL** (SG는 Deny 불가)
> - NACL은 Stateless → 응답용 임시 포트(1024~65535) 아웃바운드도 별도 허용 필요
> - SG는 Stateful → 인바운드 허용하면 응답은 자동 통과

---

### NAT Gateway vs NAT Instance

| 구분 | NAT Gateway | NAT Instance |
|------|-------------|--------------|
| 관리 주체 | AWS 완전 관리형 | EC2 직접 관리 |
| 가용성 | 고가용성 자동 | 단일 장애점 (직접 이중화 필요) |
| 확장성 | 자동 확장 | 인스턴스 타입에 제한 |
| 비용 | 상대적으로 비쌈 | 저렴 |
| Source/Dest Check | 비활성화 자동 | **수동으로 비활성화 필수** |

> ⚠️ NAT Gateway는 반드시 **퍼블릭 서브넷**에 위치해야 함

---

### Bastion Host vs Session Manager

| 구분 | Bastion Host | Systems Manager Session Manager |
|------|--------------|----------------------------------|
| SSH 키 | 필요 | 불필요 |
| 퍼블릭 IP | 필요 | 불필요 |
| 감사 로그 | 별도 설정 | CloudTrail 자동 기록 |
| 보안성 | 낮음 (포트 22 노출) | 높음 (포트 개방 불필요) |

> ⚠️ "Bastion Host 없이 프라이빗 EC2 접근" → **Systems Manager Session Manager**

---

### Route 53

| 라우팅 정책 | 설명 |
|-------------|------|
| Simple | 단순 단일 레코드 |
| Weighted | A/B 테스트 |
| Failover | Active-Passive 장애 조치 |
| Latency | 지연 최소화 |
| Geolocation | 사용자 위치 기반 |
| Geoproximity | 편향(Bias) 설정 가능 |
| Multi-value | 여러 IP 반환 |

- **Alias 레코드**: ELB, CloudFront, S3에 직접 매핑
- Health Check로 엔드포인트 상태 모니터링

---

### CloudFront

- **용도**: 글로벌 CDN, 450+ 엣지 로케이션 캐싱
- 오리진: S3, EC2, ALB, HTTP 서버
- **OAC** (Origin Access Control): S3 직접 접근 차단
- **Lambda@Edge / CloudFront Functions**: 엣지 코드 실행
- ⚠️ 정적 콘텐츠 가속 + DDoS 보호 / Price Class로 엣지 범위 제한

---

### ELB (Elastic Load Balancer)

| 유형 | 레이어 | 특징 | 사용 사례 |
|------|--------|------|-----------|
| **ALB** | L7 | HTTP/HTTPS, 경로/헤더/호스트 기반 라우팅 | 컨테이너, 마이크로서비스 |
| **NLB** | L4 | TCP/UDP, 초고성능, 고정 IP | 게임, IoT, 실시간 |
| **GWLB** | L3 | 어플라이언스 앞단 배치 | 방화벽, IDS/IPS |

> ⚠️ 고정 IP 필요 = NLB / URL 라우팅 = ALB

---

### Direct Connect vs VPN

| 구분 | Direct Connect | Site-to-Site VPN |
|------|---------------|------------------|
| 연결 방식 | 전용 물리 회선 | 인터넷 IPSec 터널 |
| 설치 시간 | 수주~수개월 | 즉시 |
| 안정성 | 높음 | 낮음 (인터넷 의존) |

> ⚠️ 즉시 필요 = VPN / 안정성+보안 = Direct Connect / 이중화 = Direct Connect + VPN 조합

---

### AWS Global Accelerator

- **용도**: AWS 글로벌 네트워크를 사용한 네트워크 가속화 및 가용성 향상
- 사용자에게 **2개의 고정 애니캐스트(Anycast) IP 주소** 제공
- 엣지 로케이션에서 AWS 내부 전용망을 통해 가장 가까운 리전의 ALB/EC2로 라우팅

| 서비스 | 주 목적 |
|--------|---------|
| **CloudFront** | 정적/동적 콘텐츠 **캐싱** (HTTP/HTTPS 기반) |
| **Global Accelerator** | 캐싱 없음. **Anycast IP 기반 TCP/UDP 네트워크 최적화** 및 고속 장애 조치 |

---

### 트래픽 흐름 요약

**인바운드 (유저 → DB)**
```
유저 브라우저 → Route 53 → CloudFront → IGW → ALB (Public) → EC2 (Private) → RDS (Private)
```

**아웃바운드 (프라이빗 EC2 → 인터넷)**
```
EC2 (Private Subnet) → NAT Gateway (Public Subnet) → IGW → 인터넷
```

---

### 3계층 Security Group 설정 예시

**웹 계층 SG (ALB)**

| 방향 | 포트 | 소스 | 설명 |
|------|------|------|------|
| 인바운드 | 80, 443 | 0.0.0.0/0 | 인터넷 전체 허용 |
| 아웃바운드 | 전체 | 0.0.0.0/0 | 기본 전체 허용 |

**앱 계층 SG (EC2)**

| 방향 | 포트 | 소스 | 설명 |
|------|------|------|------|
| 인바운드 | 8080 | ALB SG | ALB에서만 허용 |
| 인바운드 | 22 | Bastion SG | Bastion에서만 SSH |
| 아웃바운드 | 전체 | 0.0.0.0/0 | 기본 전체 허용 |

**DB 계층 SG (RDS)**

| 방향 | 포트 | 소스 | 설명 |
|------|------|------|------|
| 인바운드 | 3306 | 앱 계층 SG | 앱 서버에서만 허용 |
| 아웃바운드 | 전체 | 0.0.0.0/0 | 기본 전체 허용 |

> ⚠️ **SG 체이닝**: 소스를 IP 대신 **다른 SG ID**로 지정하는 것이 Best Practice

---

### 시험 기출 Q&A

| 질문 | 답변 |
|------|------|
| 프라이빗 서브넷 EC2가 인터넷에 접근해야 한다면? | 퍼블릭 서브넷에 NAT GW 생성 + 프라이빗 라우팅 테이블에 `0.0.0.0/0 → NAT GW` 추가 |
| 특정 IP를 완전히 차단하려면? | NACL에 Deny 규칙 추가 (SG는 Deny 불가 → 오답) |
| ALB는 어느 서브넷에 배치? | 퍼블릭 서브넷 |
| RDS 외부 접근 차단 방법은? | 프라이빗 서브넷 배치 + 라우팅에 0.0.0.0/0 없음 + SG에서 앱 계층 SG만 인바운드 허용 |
| 관리자가 프라이빗 EC2에 SSH 접근하려면? | Bastion Host(퍼블릭 서브넷) 경유 또는 Systems Manager Session Manager 사용 |

---

## 🔐 보안 / IAM

### IAM

- User, Group, Role, Policy 구성
- **최소 권한 원칙** (Least Privilege) 필수
- **Role**: EC2, Lambda 등 서비스에 임시 권한 부여
- **STS AssumeRole**: 교차 계정 접근
- Identity-based Policy vs Resource-based Policy
- ⚠️ MFA 활성화 필수 / 루트 계정 일상 사용 금지

---

### KMS (Key Management Service)

- **CMK**: AWS Managed / Customer Managed
- 자동 키 순환(Automatic Rotation) 지원
- CloudTrail로 키 사용 감사
- ⚠️ 대부분 AWS 서비스 암호화 = KMS 연동

---

### WAF & Shield & ACM

| 서비스 | 설명 |
|--------|------|
| **WAF** | L7 공격 차단 (SQL 인젝션, XSS) → ALB, CloudFront, API GW 적용 |
| **Shield Standard** | 자동 적용, L3/L4 DDoS 방어 (무료) |
| **Shield Advanced** | 유료($3,000/월). L7 DDoS 방어, DRT 24/7 지원, 비용 보상 포함 |
| **ACM** | SSL/TLS 인증서 프로비저닝 및 자동 갱신. CloudFront, ALB 연동 |

> ⚠️ DDoS 방어 = Shield / 웹 공격 차단 = WAF

---

### 보안 진단 서비스 비교

| 서비스 | 용도 | 시험 키워드 |
|--------|------|-------------|
| **CloudTrail** | AWS API 호출 기록 (누가, 언제, 무엇을) | "누가 리소스를 생성했는가?" 감사 |
| **AWS Config** | 리소스 구성 변경 이력 추적 및 규정 준수 검사 | "리소스 설정이 어떻게 변경되었는가?" |
| **Amazon Inspector** | EC2, ECR 컨테이너, Lambda 보안 취약점 점검 | "EC2 내부 패치/OS 취약점 스캔" |
| **Amazon GuardDuty** | 지능형 위협 탐지 (VPC 흐름 로그, DNS, CloudTrail 분석) | "계정 인프라 전체 위협/공격징후 탐지" |
| **Amazon Macie** | S3 내 민감 데이터(PII) 자동 발견 및 보호 | "S3 버킷 내 개인정보(PII) 식별" |

---

### Secrets Manager vs Parameter Store

| 구분 | Secrets Manager | Parameter Store |
|------|-----------------|-----------------|
| 비용 | 유료 | 무료 (Standard) |
| 자동 교체 | ✅ 지원 | ❌ 미지원 |
| 용도 | DB 비밀번호, API 키 | 설정값, 환경변수 |

---

## 📊 분석 / ML

### Kinesis

| 서비스 | 특징 | 사용 사례 |
|--------|------|-----------|
| **Kinesis Data Streams** | 실시간 스트림, 샤드 기반, 보존 최대 365일. 생산자/소비자 직접 구현 필요 | 커스텀 실시간 분석/처리 |
| **Kinesis Data Firehose** | Near Real-time (최소 60초 버퍼). S3/Redshift/Splunk로 코드 없이 자동 전달 | 데이터를 S3/Redshift에 적재만 할 때 |
| **Kinesis Data Analytics** | SQL로 실시간 분석 | 스트림 실시간 SQL 분석 |

> ⚠️ 실시간 처리 = Kinesis / 배치 = SQS

---

### Athena

- S3의 데이터를 직접 SQL로 쿼리 (서버 불필요)
- 과금: 스캔 데이터 1TB당 $5
- Parquet/ORC 형식으로 비용 절감
- ⚠️ 로그 분석, S3 임시 쿼리에 최적

---

### Glue

- 서버리스 ETL 서비스
- **Glue Data Catalog**: 메타데이터 카탈로그 (Athena와 통합)
- **Glue Crawler**: 데이터 스키마 자동 탐색
- ⚠️ S3 → Redshift 데이터 변환에 자주 사용

---

### Amazon SageMaker

- **용도**: ML 모델 구축, 훈련 및 배포를 위한 완전 관리형 서비스
- 데이터 전처리, 자동 모델 튜닝(HPO), 엔드포인트 호스팅까지 ML 파이프라인 전체 지원

---

## 🔗 통합 / 메시지

### SQS (Simple Queue Service)

| 큐 유형 | 특징 |
|---------|------|
| **표준 큐** | 최소 1회 전달, 순서 보장 X |
| **FIFO 큐** | 정확히 1회 전달, 순서 보장 (300 TPS) |

- 메시지 보존: 최대 14일 / 크기: 최대 256KB
- **Visibility Timeout**: 처리 중 다른 소비자에게 숨김
- **DLQ** (Dead Letter Queue): 처리 실패 메시지 보관
- ⚠️ 디커플링, 버퍼링에 핵심

---

### SNS (Simple Notification Service)

- 토픽에 메시지 발행 → 여러 구독자에게 팬아웃
- 구독: Lambda, SQS, HTTP, Email, SMS
- FIFO SNS 지원 (순서 보장)
- ⚠️ **SNS + SQS 팬아웃 패턴** 자주 출제

---

### EventBridge

- AWS 서비스 이벤트를 Lambda, SQS, SNS 등에 라우팅
- 규칙(Rule) 기반 이벤트 필터링
- Cron/Rate 스케줄 지원
- ⚠️ CloudWatch Events의 업그레이드 버전

---

### Step Functions

- Lambda 함수 체이닝, 병렬 처리, 조건 분기
- **Express Workflow**: 고속 단발성 / **Standard**: 장기 실행
- ⚠️ Lambda 15분 제한 초과하는 복잡한 흐름에 사용

---

## 🚚 마이그레이션

### Snowball Edge

| 유형 | 특징 |
|------|------|
| **Storage Optimized** | 80TB 스토리지 |
| **Compute Optimized** | EC2/Lambda 엣지 컴퓨팅 + 28TB |

- **Snowmobile**: 100PB 이상 극대용량
- ⚠️ 네트워크 느리거나 대용량(>10TB) 이전 시 Snowball 선택

---

### DMS (Database Migration Service)

- 동종/이기종 DB 마이그레이션 (Oracle → Aurora 등)
- **CDC** (Change Data Capture): 무중단 마이그레이션
- **SCT** (Schema Conversion Tool): 스키마 변환
- ⚠️ 동종 = 직접 이전 / 이기종 = SCT 후 DMS

---

### AWS Backup

- EC2, EBS, RDS, DynamoDB, EFS, S3 등 통합 백업
- 백업 플랜, 보존 정책, 크로스 리전/계정 복사
- ⚠️ 다수 서비스 백업 중앙 관리 = AWS Backup

---

### 데이터 이동 서비스 비교

| 서비스 | 용도 | 시험 키워드 |
|--------|------|-------------|
| **AWS DataSync** | 온프레미스 ↔ AWS 스토리지 온라인 고속 전송 (에이전트 설치) | "인터넷 충분, 지속적 동기화 필요" |
| **Snowball Edge** | 물리 장치로 오프라인 대용량 이전 | "인터넷 불가, 대용량 물리 이전" |
| **AWS Transfer Family** | SFTP/FTPS/FTP 프로토콜로 S3/EFS 파일 전송 | "기존 SFTP 워크플로우 유지" |
| **Amazon AppFlow** | SaaS(Salesforce, Slack 등) ↔ AWS 데이터 연동 (No-Code ETL) | "Salesforce 데이터를 S3로 코딩 없이 연동" |

---

## ⚡ 핵심 선택 가이드

### 스토리지 선택

| 상황 | 선택 |
|------|------|
| 객체 저장 (이미지, 파일) | S3 |
| EC2 전용 블록 스토리지 | EBS |
| 여러 EC2가 공유하는 파일 시스템 (Linux) | EFS |
| Windows EC2 공유 파일 시스템 | FSx for Windows |
| HPC / ML 고성능 파일 시스템 | FSx for Lustre |
| 대용량 오프라인 데이터 이전 | Snowball Edge |
| 온프레미스 ↔ AWS 파일 연동 | Storage Gateway |

### DB 선택

| 상황 | 선택 |
|------|------|
| 관계형 DB (범용) | RDS |
| 관계형 DB (고성능/고가용성) | Aurora |
| NoSQL, 밀리초 응답 | DynamoDB |
| 인메모리 캐시 | ElastiCache (Redis/Memcached) |
| 데이터 웨어하우스 / 분석 | Redshift |

### 고가용성 vs 성능

| 목적 | 방법 |
|------|------|
| 장애 조치 (자동 복구) | Multi-AZ |
| 읽기 성능 향상 | Read Replica |
| 글로벌 재해 복구 | Aurora Global Database |

### 메시지 / 통합

| 상황 | 선택 |
|------|------|
| 서비스 간 디커플링, 버퍼링 | SQS |
| 여러 서비스에 동시 알림 (팬아웃) | SNS → SQS |
| 이벤트 기반 자동화 | EventBridge |
| 복잡한 워크플로우 | Step Functions |

### 네트워크

| 상황 | 선택 |
|------|------|
| URL 경로 기반 라우팅 | ALB |
| 고정 IP, 초고성능 TCP | NLB |
| 전용선 연결 | Direct Connect |
| 즉시 VPN 연결 | Site-to-Site VPN |
| 인터넷 없이 AWS 서비스 접근 | VPC Endpoint |
| 고정 Anycast IP 기반 글로벌 가속 | Global Accelerator |

---

## 💡 최종 암기 요약

### 네트워킹
- 퍼블릭 여부 = 라우팅 테이블의 **IGW 유무**
- 특정 IP 차단 = **NACL** (Deny 가능)
- 프라이빗 아웃바운드 = **NAT Gateway** (퍼블릭 서브넷에 위치)
- SG = **Stateful** / NACL = **Stateless**
- Bastion 없이 SSH = **Session Manager**

### 보안
- API 호출 로깅 = **CloudTrail**
- 리소스 설정 추적 = **AWS Config**
- EC2 취약점 스캔 = **Inspector**
- 위협 탐지 = **GuardDuty**
- S3 PII 식별 = **Macie**
- 웹 공격 차단 = **WAF**
- 대규모 DDoS = **Shield Advanced**

### 데이터 이동
- 온라인 고속 동기화 = **DataSync**
- 기존 SFTP 유지 = **Transfer Family**
- SaaS 연동 = **AppFlow**
- 오프라인 대용량 = **Snowball Edge**