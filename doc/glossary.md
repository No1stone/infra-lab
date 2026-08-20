# 플랫폼 컴포넌트 용어집

> **용어 안내** 
> 이 도구들을 흔히 **서드파티(third-party)** 라고도 부르지만, 더 정확히는 **오픈소스, CNCF 플랫폼 컴포넌트**(또는 클라우드 관리형과 대응하는 **자가 호스팅 스택**)다. 
> **서드파티** = “우리가 직접 안 만든 외부 소프트웨어”라는 넓은 말. 틀리진 않음. 랩/문서에서는 **플랫폼 컴포넌트**를 기본 용어로 쓴다.

- 아키텍처 기준: [`ref/Ref.md`](../ref/Ref.md)
- 공식 문서 URL: [`official-docs.md`](official-docs.md)
- 실행 명령: [`cli/ops/`](../cli/ops/)
- DNS: 랩 호스트는 **3뎁스만** — 예) `echo.nginx.lab.origemite.com` (`<앱>.<게이트웨이>.lab.origemite.com`). 2뎁스 `*.lab.origemite.com` 앱 호스트는 쓰지 않는다.

---

## 런타임, 클러스터

| 컴포넌트 | 역할 (1줄) | 대중적 대응/비유 | 랩 경로 |
| --- | --- | --- | --- |
| **k3d** | Docker 위에 k3s 클러스터를 빠르게 띄우는 랩용 도구 | kind / minikube 계열 로컬 k8s | [`cli/doc/k3d.md`](../cli/doc/k3d.md), [`doc/02-k3d-cluster.md`](02-k3d-cluster.md) |
| **Kubernetes** | 컨테이너 오케스트레이션 표준 플랫폼 | EKS / GKE / AKS | k3d 클러스터 `lab` |
| **k3s** | 경량 Kubernetes 배포판 (k3d가 래핑) | 관리형 k8s의 “작은 엔진” | k3d 이미지, 런타임 |
| **Terraform** | 선언적 IaC — 클러스터, 부트스트랩 자원을 코드로 관리 | AWS CloudFormation / Pulumi | [`terraform/`](../terraform/), [`doc/03-terraform.md`](03-terraform.md), [`cli/ops/terraform.md`](../cli/ops/terraform.md) |
| **Helm** | Kubernetes 패키지 매니저 (차트, values 배포) | k8s의 apt/yum | [`helm/values/`](../helm/values/), [`cli/doc/helm.md`](../cli/doc/helm.md) |

---

## 진입, 네트워킹

| 컴포넌트 | 역할 (1줄) | 대중적 대응/비유 | 랩 경로 |
| --- | --- | --- | --- |
| **MetalLB** | 베어메탈, 온프레 k8s에 `LoadBalancer` IP 할당 | AWS NLB/CLB 자리 (클라우드 LB 없을 때) | [`helm/values/metallb.yaml`](../helm/values/metallb.yaml), [`k8s/configmap/metallb-ip-pool.yaml`](../k8s/configmap/metallb-ip-pool.yaml) |
| **ingress-nginx** | **Ingress** 리소스 구현 — HTTP(S) L7 라우팅, TLS 종료 | AWS ALB Ingress Controller / 예전 de facto Ingress | [`helm/values/nginx.yaml`](../helm/values/nginx.yaml), [`k8s/namespace/nginx.yaml`](../k8s/namespace/nginx.yaml) |
| **Kong** | **Ingress/API Gateway** 클래스 — 플러그인, 정책 기반 L7 (nginx 웹서버와 별개) | AWS API Gateway + Ingress / Kong Gateway | [`helm/values/kong.yaml`](../helm/values/kong.yaml), [`k8s/ingress/demo-kong.yaml`](../k8s/ingress/demo-kong.yaml) |
| **Traefik** | Ingress 컨트롤러 — 동적 설정, Let's Encrypt 연동에 강함 | nginx Ingress / HAProxy Ingress 동급 | [`helm/values/traefik.yaml`](../helm/values/traefik.yaml), [`k8s/ingress/demo-traefik.yaml`](../k8s/ingress/demo-traefik.yaml) |
| **HAProxy Ingress** | HAProxy 기반 Ingress 컨트롤러 — 고성능 L7 | nginx Ingress / Traefik 동급 | [`helm/values/haproxy-ingress.yaml`](../helm/values/haproxy-ingress.yaml), [`k8s/ingress/demo-haproxy.yaml`](../k8s/ingress/demo-haproxy.yaml) |
| **Envoy Gateway** | **Gateway API** 구현 — Envoy 데이터 플레인 | ALB/NLB + Gateway API (차세대) | [`helm/values/envoy-gateway.yaml`](../helm/values/envoy-gateway.yaml), [`k8s/gateway/demo-envoy.yaml`](../k8s/gateway/demo-envoy.yaml) |
| **Cilium Gateway** | Cilium CNI 위 **Gateway API** 구현 | Gateway API + eBPF 네트워킹 | [`helm/values/cilium.yaml`](../helm/values/cilium.yaml), [`k8s/gateway/demo-cilium.yaml`](../k8s/gateway/demo-cilium.yaml) |
| **Istio Gateway** | Istio 메시와 연동된 **Gateway API** 구현 | App Mesh / Gateway API + mTLS | [`helm/values/istio-gateway.yaml`](../helm/values/istio-gateway.yaml), [`k8s/gateway/demo-istio.yaml`](../k8s/gateway/demo-istio.yaml) |
| **Gateway API** | Ingress 후속 Kubernetes 표준 (Gateway, HTTPRoute 등) | Ingress v2 / 차세대 L7 API | [`doc/compare/gateway-compare.md`](compare/gateway-compare.md), [`cli/ops/gateway-api-features.md`](../cli/ops/gateway-api-features.md) |

---

## CNI, 보안, 메시

| 컴포넌트 | 역할 (1줄) | 대중적 대응/비유 | 랩 경로 |
| --- | --- | --- | --- |
| **Cilium** | eBPF 기반 CNI + NetworkPolicy + Hubble 관측 | Calico + 관측 강화 / AWS VPC CNI + 정책 | [`helm/values/cilium.yaml`](../helm/values/cilium.yaml), [`doc/09-cilium.md`](09-cilium.md) |
| **CiliumNetworkPolicy** | L3–L7 파드, FQDN 간 통신 허용/거부 | Security Group + NACL (파드 단위) | [`k8s/ciliumnetworkpolicy/`](../k8s/ciliumnetworkpolicy/) |
| **Istio** | 서비스 메시 — 트래픽, 보안, 관측 sidecar/ambient | AWS App Mesh / Linkerd 동급 | [`helm/values/istiod.yaml`](../helm/values/istiod.yaml), [`doc/10-istio-kiali.md`](10-istio-kiali.md) |
| **PeerAuthentication (mTLS)** | east-west 상호 TLS 모드 (STRICT/PERMISSIVE 등) | mTLS between services / SPIFFE-like | [`k8s/peerauthentication/istio-mtls.yaml`](../k8s/peerauthentication/istio-mtls.yaml), [`doc/11-mtls.md`](11-mtls.md) |
| **Kiali** | 서비스 메시 토폴로지, 트래픽 UI | App Mesh console / Datadog service map (메시) | [`helm/values/kiali.yaml`](../helm/values/kiali.yaml) |
| **cert-manager** | TLS 인증서 자동 발급, 갱신 (ACME/Let's Encrypt) | ACM + 자동 갱신 / Let's Encrypt operator | [`helm/values/cert-manager.yaml`](../helm/values/cert-manager.yaml), [`k8s/configmap/cert-manager-clusterissuer.yaml`](../k8s/configmap/cert-manager-clusterissuer.yaml) |

---

## IAM, 시크릿, 레지스트리

| 컴포넌트 | 역할 (1줄) | 대중적 대응/비유 | 랩 경로 |
| --- | --- | --- | --- |
| **Keycloak** | OIDC/OAuth2 IdP — 사용자, 클라이언트, 역할 관리 | AWS Cognito / Okta / Auth0 | [`helm/values/keycloak.yaml`](../helm/values/keycloak.yaml), [`doc/13-keycloak.md`](13-keycloak.md) |
| **Vault** | 시크릿, 동적 자격증명, 암호화 키 저장 | AWS Secrets Manager / SSM Parameter Store / Azure Key Vault | [`k8s/deployment/vault.yaml`](../k8s/deployment/vault.yaml), [`k8s/namespace/vault.yaml`](../k8s/namespace/vault.yaml) |
| **Harbor** | 컨테이너 이미지 사내 레지스트리 + 스캔, 서명 | Amazon ECR / GHCR / Docker Hub (프라이빗) | [`helm/values/harbor.yaml`](../helm/values/harbor.yaml), [`doc/12-harbor.md`](12-harbor.md) |

---

## GitOps, 배포

| 컴포넌트 | 역할 (1줄) | 대중적 대응/비유 | 랩 경로 |
| --- | --- | --- | --- |
| **Argo CD** | Git 저장소를 단일 진실원으로 하는 선언적 CD | Flux CD 동급 / CodePipeline (GitOps) | [`helm/values/argocd.yaml`](../helm/values/argocd.yaml), [`argocd/application/`](../argocd/application/), [`doc/07-argocd-headlamp.md`](07-argocd-headlamp.md) |
| **Argo Rollouts** | Blue/Green, Canary 등 고급 배포 전략 CRD | Flagger / Spinnaker canary | [`k8s/rollout/`](../k8s/rollout/), [`cli/ops/argo-rollouts.md`](../cli/ops/argo-rollouts.md) |
| **Headlamp** | 웹 기반 Kubernetes 클러스터 UI | Lens / Kubernetes Dashboard | [`helm/values/headlamp.yaml`](../helm/values/headlamp.yaml) |

---

## 관측

| 컴포넌트 | 역할 (1줄) | 대중적 대응/비유 | 랩 경로 |
| --- | --- | --- | --- |
| **Prometheus** | 메트릭 수집, 알림 (pull 기반 TSDB) | CloudWatch Metrics / Datadog metrics backend | [`helm/values/kube-prometheus-stack.yaml`](../helm/values/kube-prometheus-stack.yaml) |
| **Grafana** | 메트릭, 로그, 트레이스 대시보드 UI | CloudWatch Dashboards / Datadog UI | 동일 (`kube-prometheus-stack`) |
| **Loki** | 라벨 기반 로그 집계, 검색 (Prometheus식) | CloudWatch Logs (라벨 중심) / Splunk 경량 | [`helm/values/loki.yaml`](../helm/values/loki.yaml) |
| **Fluent Bit** | 노드, 컨테이너 로그 수집, 전달 에이전트 | CloudWatch Logs agent / Fluentd 경량 | [`helm/values/fluent-bit.yaml`](../helm/values/fluent-bit.yaml) |
| **Tempo** | 분산 트레이스 저장, 쿼리 (OTLP 수신) | X-Ray / Jaeger / Zipkin 대체 축 | [`helm/values/tempo.yaml`](../helm/values/tempo.yaml), [`cli/ops/otel-trace-lab.md`](../cli/ops/otel-trace-lab.md) |
| **OpenTelemetry Collector** | 메트릭, 로그, 트레이스 수집, 변환, 라우팅 파이프 | CloudWatch Agent + OTel / vendor-neutral pipeline | [`helm/values/opentelemetry-collector.yaml`](../helm/values/opentelemetry-collector.yaml) |
| **OpenSearch** | 로그, 이벤트 전문 검색, 대시보드 (선택) | Elasticsearch + Kibana / CloudWatch Logs Insights | [`helm/values/opensearch.yaml`](../helm/values/opensearch.yaml), [`doc/08-opensearch.md`](08-opensearch.md) |

---

## 데이터

| 컴포넌트 | 역할 (1줄) | 대중적 대응/비유 | 랩 경로 |
| --- | --- | --- | --- |
| **MySQL** | 관계형 DB (OLTP) | Amazon RDS MySQL / Aurora MySQL | [`k8s/deployment/mysql.yaml`](../k8s/deployment/mysql.yaml), [`k8s/node/mysql/`](../k8s/node/mysql/) |
| **Redis** | 인메모리 KV, 캐시, Pub/Sub | Amazon ElastiCache Redis | [`k8s/deployment/redis.yaml`](../k8s/deployment/redis.yaml), [`k8s/node/redis/`](../k8s/node/redis/) |
| **Kafka** | 분산 이벤트 스트리밍, 로그 버스 | Amazon MSK / Confluent Cloud | [`k8s/deployment/kafka.yaml`](../k8s/deployment/kafka.yaml), [`k8s/node/kafka/`](../k8s/node/kafka/) |
| **RabbitMQ** | AMQP 메시지 브로커 (큐, 라우팅) | Amazon MQ (RabbitMQ) / SQS (패턴 유사) | [`k8s/deployment/rabbitmq.yaml`](../k8s/deployment/rabbitmq.yaml), [`k8s/node/rabbitmq/`](../k8s/node/rabbitmq/) |

---

## 기타 랩 맥락

| 컴포넌트 | 역할 (1줄) | 대중적 대응/비유 | 랩 경로 |
| --- | --- | --- | --- |
| **AWS 프록시 + reverse SSH** | 홈랩 공개 입구 — Ubuntu 노트북으로 터널 (ALB 앞단 아님) | ngrok / Cloudflare Tunnel / bastion 패턴 | [`cli/doc/proxy.md`](../cli/doc/proxy.md), [`cli/ops/proxy.md`](../cli/ops/proxy.md), [`doc/01-proxy-and-dns.md`](01-proxy-and-dns.md) |
| **Gitea** *(언급만)* | 개인 Git 호스팅 (infra-lab 외부) | GitHub (self-hosted) | 외부 인프라 |
| **Jenkins** *(언급만)* | 개인 CI (infra-lab 외부) | GitHub Actions / GitLab CI | 외부 인프라 |

---

## Kubernetes 운영 기능 (내장, 부가)

서드파티가 아니라 **쿠버네티스/애드온**에 가깝다.

| 컴포넌트 | 역할 (1줄) | 대중적 대응/비유 | 랩 경로 |
| --- | --- | --- | --- |
| **PDB** (PodDisruptionBudget) | drain 등 voluntary 중단 시 최소 가용 Pod 보장 | ASG 최소 인스턴스 / 무중단 유지보수 가드레일 | [`cli/ops/pdb.md`](../cli/ops/pdb.md), [`k8s/poddisruptionbudget/`](../k8s/poddisruptionbudget/) |
| **HPA** | CPU 등 메트릭 기반 Pod 자동 스케일 | ASG / App Auto Scaling | [`cli/ops/hpa.md`](../cli/ops/hpa.md), [`k8s/horizontalpodautoscaler/`](../k8s/horizontalpodautoscaler/) |
| **metrics-server** | 리소스 메트릭 API (HPA 입력) | CloudWatch agent (노드, 파드 CPU) 일부 역할 | [`cli/ops/hpa.md`](../cli/ops/hpa.md) |

---

## 관련 비교 문서

| 주제 | 문서 |
| --- | --- |
| Ingress vs Gateway API | [`compare/gateway-compare.md`](compare/gateway-compare.md), [`14-ingress-compare.md`](14-ingress-compare.md) |
| CNI | [`compare/cni-compare.md`](compare/cni-compare.md) |
| Service mesh | [`compare/service-mesh-compare.md`](compare/service-mesh-compare.md) |
