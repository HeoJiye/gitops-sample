# GitOps Sample

로컬에서 사용하는 GitOps 학습용 레포

## 구조

```
gitops-sample/
├── applications/           # 애플리케이션별 헬름차트
│   └── spring-boot-sample/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
├── argocd/                # ArgoCD 애플리케이션 정의
│   └── applications/
│       └── spring-boot-sample.yaml
├── environments/          # 환경별 설정
│   ├── dev/
│   ├── staging/
│   └── prod/
└── README.md
```

## 사용 방법

### 1. ArgoCD 애플리케이션 배포

```bash
# ArgoCD 애플리케이션 등록
kubectl apply -f argocd/applications/spring-boot-sample.yaml
```

### 2. 환경별 배포

#### 로컬 환경 (k3d)
```bash
# 로컬 이미지 빌드 및 k3d 클러스터로 가져오기
docker build -t heojiye/springboot-sample:latest -f docker/Dockerfile ../springboot-sample
k3d image import heojiye/springboot-sample:latest -c your-cluster-name

# 로컬 환경 배포
helm upgrade --install spring-boot-sample-local ./applications/spring-boot-sample \
  -f environments/local/values.yaml \
  -n local --create-namespace

# 애플리케이션 확인 (NodePort 사용)
curl http://localhost:30080/actuator/health
```

#### 개발 환경
```bash
helm upgrade --install spring-boot-sample-dev ./applications/spring-boot-sample \
  -f environments/dev/values.yaml \
  -n dev --create-namespace
```

#### 스테이징 환경
```bash
helm upgrade --install spring-boot-sample-staging ./applications/spring-boot-sample \
  -f environments/staging/values.yaml \
  -n staging --create-namespace
```

#### 프로덕션 환경
```bash
helm upgrade --install spring-boot-sample-prod ./applications/spring-boot-sample \
  -f environments/prod/values.yaml \
  -n prod --create-namespace
```

## 관련 레포

- **GitOps 레포**: https://github.com/HeoJiye/gitops-sample
- **스프링부트 애플리케이션 레포**: https://github.com/HeoJiye/springboot-sample

## Docker 이미지 빌드 및 푸시

### 1. 스프링부트 애플리케이션 이미지 빌드

```bash
# 스프링부트 애플리케이션 레포로 이동
cd ../springboot-sample

# Docker 이미지 빌드
docker build -t heojiye/springboot-sample:latest -f ../gitops-sample/docker/Dockerfile .

# Docker Hub에 푸시 (선택사항)
docker push heojiye/springboot-sample:latest
```

### 2. 로컬 테스트

```bash
# docker-compose로 로컬 테스트
cd docker/
docker-compose up -d

# 애플리케이션 확인
curl http://localhost:8080/actuator/health
```

## K3d 클러스터에서 로컬 이미지 사용

### 1. 로컬 이미지를 k3d 클러스터로 가져오기

```bash
# 이미지 빌드 후 k3d 클러스터로 가져오기
k3d image import heojiye/springboot-sample:latest -c your-cluster-name

# 또는 이미지 빌드와 동시에 가져오기
docker build -t heojiye/springboot-sample:latest -f docker/Dockerfile ../springboot-sample
k3d image import heojiye/springboot-sample:latest -c your-cluster-name
```

### 2. 로컬 이미지 사용 시 헬름차트 수정

로컬에서 이미지를 사용할 때는 `imagePullPolicy`를 `Never`로 설정하여 레지스트리에서 이미지를 가져오지 않도록 합니다:

```yaml
# values.yaml 또는 환경별 values.yaml
image:
  repository: heojiye/springboot-sample
  tag: "latest"
  pullPolicy: Never  # 로컬 이미지 사용 시
```

## 주의사항

1. 각 환경별로 적절한 리소스 설정을 확인하세요.
2. Docker 이미지를 먼저 빌드하고 레지스트리에 푸시하거나 k3d 클러스터로 가져와야 합니다.
3. 로컬 개발 시에는 `imagePullPolicy: Never`를 사용하여 로컬 이미지를 사용하세요.

## 환경별 특징

- **local**: 로컬 k3d 클러스터용, NodePort 서비스, 로컬 이미지 사용
- **dev**: 최소 리소스, 단일 레플리카
- **staging**: 중간 리소스, 2개 레플리카, ingress 활성화
- **prod**: 최대 리소스, 3개 레플리카, 오토스케일링 활성화
