# infra-lab

홈랩에서 Kubernetes 운영 스택을 연습하는 저장소입니다.

기준은 [2026년 쿠버네티스 표준 아키텍처](ref/Ref.md)입니다. 런타임은 Ubuntu 노트북의 **k3d**이고, 공개 호스트는 **3뎁스** `<앱>.<게이트웨이>.lab.origemite.com`만 씁니다 (2뎁스 `*.lab.origemite.com` 앱 호스트는 쓰지 않음).

## 자원

| 역할 | 머신 | 스펙 |
| --- | --- | --- |
| 보유 | MacBook Pro | macOS |
| 보유 | MacBook Air | macOS |
| 공개 입구 | AWS 프록시 | Ubuntu, 1 vCPU / 2 GiB |
| 랩 호스트 | Ubuntu 노트북 | Intel 8세대 i5, RAM 32 GiB, swap 64 GiB |

## 연결

```text
인터넷
  └─ *.nginx|gateway|cilium|… .lab.origemite.com   (3뎁스만)
       └─ AWS 프록시 (Host → 게이트웨이별 upstream)
            └─ reverse SSH (80/443 + 8201–8207)
                 └─ Ubuntu 노트북 (k3d / MetalLB .201–.207)
```

예: `argocd.nginx.lab.origemite.com`, `rabbitmq.nginx.lab.origemite.com`, `demo.istio.lab.origemite.com`.

1. Ubuntu 노트북이 AWS 프록시로 reverse SSH를 유지합니다.
2. DNS: `origemite.com`은 개인용(이 랩에서 안 건드림). 이름만 [`dns/inventory.yaml`](dns/inventory.yaml).
3. 프록시가 Host로 분기해 MetalLB `.201`–`.207`에 넘깁니다([`cli/ops/proxy.md`](cli/ops/proxy.md)).

## 커리큘럼

클러스터 이름 `lab`. 서버 1 + 에이전트 5. 데이터 워크로드는 **1 노드 = 파드 1개** (`lab.origemite.com/workload=<이름>`).

| 단계 | 내용 | 경로 |
| --- | --- | --- |
| 0 | k3d 클러스터 생성, 노드 라벨 | [`cli/ops/k3d.md`](cli/ops/k3d.md), [`cli/ops/kube.md`](cli/ops/kube.md) |
| 0T | **Terraform** — k3d 부트스트랩 (**Experimental**→최소 구현) | [`terraform/`](terraform/), [`cli/ops/terraform.md`](cli/ops/terraform.md) |
| 1 | MetalLB → ingress-nginx → cert-manager | [`helm/values/`](helm/values/), [`cli/ops/helm.md`](cli/ops/helm.md) |
| 2 | 데이터 워크로드 (raw k8s) | [`k8s/`](k8s/) |
| 3 | 관측: Prometheus/Grafana, Loki, Tempo, OTel, Fluent Bit | [`helm/values/`](helm/values/) |
| 3T | **분산 트레이스 랩** (Gateway→앱→Redis/Kafka→Tempo) | [`cli/ops/otel-trace-lab.md`](cli/ops/otel-trace-lab.md) |
| 4 | GitOps·UI: Argo CD, Headlamp | [`helm/values/argocd.yaml`](helm/values/argocd.yaml) |
| 4G | **GitOps** — Application / Auto Sync / Self Heal / Drift | [`cli/ops/argocd-gitops.md`](cli/ops/argocd-gitops.md), [`argocd/application/`](argocd/application/) |
| 4R | **Argo Rollouts** — Blue/Green·Canary | [`cli/ops/argo-rollouts.md`](cli/ops/argo-rollouts.md) |
| 5 | (선택) OpenSearch | [`helm/values/opensearch.yaml`](helm/values/opensearch.yaml) |
| 6A | Cilium — NetworkPolicy·L7/DNS·Hubble | [`cli/ops/cilium.md`](cli/ops/cilium.md), [`k8s/ciliumnetworkpolicy/`](k8s/ciliumnetworkpolicy/) |
| 6B | Istio + Kiali | [`cli/ops/istio.md`](cli/ops/istio.md) |
| 6B-m | **mTLS** (Istio STRICT; 타 스택은 비교 문서) | [`cli/ops/mtls.md`](cli/ops/mtls.md), [`doc/compare/service-mesh-compare.md`](doc/compare/service-mesh-compare.md) |
| 6C | Harbor | [`cli/ops/harbor.md`](cli/ops/harbor.md) |
| 6D | **Keycloak** (필수) | [`cli/ops/keycloak.md`](cli/ops/keycloak.md) |
| 7 | 진입점 비교 (Ingress/Gateway 7종) | [`cli/ops/ingress-compare.md`](cli/ops/ingress-compare.md) |
| 7F | Gateway API **기능 체크리스트** (gRPC·JWT 등) | [`cli/ops/gateway-api-features.md`](cli/ops/gateway-api-features.md), [`doc/compare/gateway-compare.md`](doc/compare/gateway-compare.md) |
| 8O | **운영** — PDB / HPA / Node failure | [`cli/ops/pdb.md`](cli/ops/pdb.md), [`cli/ops/hpa.md`](cli/ops/hpa.md), [`cli/ops/node-failure.md`](cli/ops/node-failure.md) |
| 8C | **Chaos** — Pod/Node/Gateway/데이터/정책 장애 | [`cli/ops/chaos.md`](cli/ops/chaos.md) |
| S | **운영 시나리오** (무중단·DR·인증서·게이트웨이 이전 등) | [`doc/scenarios/`](doc/scenarios/) |
| C | **비교 문서** (Gateway / CNI / Mesh) | [`doc/compare/`](doc/compare/) |
| 이후 | Keycloak→Argo/Kiali OIDC | `argocd/` |
| 별도 | **Kubeflow** — 이 랩에 올리지 않음 | 별도 머신 |

실습 흐름: **설치 → 운영(PDB/HPA/장애) → GitOps/점진배포 → 시나리오**. 비교 문서로 선택지를 정리한다.

## 튜토리얼·운영 문서

| 경로 | 역할 |
| --- | --- |
| [`doc/README.md`](doc/README.md) | 설치·단계 튜토리얼 |
| [`doc/scenarios/`](doc/scenarios/) | **운영 시나리오** (무중단·DR·인증서 등) |
| [`doc/compare/`](doc/compare/) | Gateway / CNI / Mesh **비교** |

**Kubeflow**는 모니터링이 아니라 ML 파이프라인용입니다. 메모리 부담이 커서 **infra-lab(Ubuntu k3d)과 리소스를 분리**하고, ML 전용 호스트/클러스터에 할당합니다.

## 스택

### 진입점

- 네임스페이스 `nginx`, Ingress class `nginx` (플랫폼 기본 게이트웨이)
- 호스트 **3뎁스만**: `<앱>.nginx.lab.origemite.com` (예: `argocd`, `grafana`, `vault`, `rabbitmq`, `headlamp`)
- 다른 게이트웨이로 붙일 때는 `<앱>.<게이트웨이>.lab.origemite.com`
- 2뎁스(`rabbitmq.lab.origemite.com` 등)는 쓰지 않는다
- ingress-nginx는 학습용. 2026 표준상 EOL 이후 Gateway API 전환을 염두에 둡니다.

## 진입점 비교 (Phase 7)

7개 컨트롤러가 공유 데모 앱 [`demo-echo`](k8s/deployment/demo-echo.yaml)로 동일 Host 라우팅을 검증한다. MetalLB IP `.201`–`.207`에 컨트롤러별 LoadBalancer를 고정한다.

| 호스트 | API | 컨트롤러 | MetalLB IP |
| --- | --- | --- | --- |
| demo.nginx.lab.origemite.com | Ingress | ingress-nginx | 172.18.255.201 |
| demo.gateway.lab.origemite.com | Gateway API | Envoy Gateway | 172.18.255.202 |
| demo.cilium.lab.origemite.com | Gateway API | Cilium Gateway | 172.18.255.203 |
| demo.kong.lab.origemite.com | Ingress | Kong | 172.18.255.204 |
| demo.traefik.lab.origemite.com | Ingress | Traefik | 172.18.255.205 |
| demo.istio.lab.origemite.com | Gateway API | Istio Gateway | 172.18.255.206 |
| demo.haproxy.lab.origemite.com | Ingress | HAProxy Ingress | 172.18.255.207 |

- **동시 기동**: idle 기준 RAM 약 **+4GiB** (32GiB+swap64에서 Phase 1–6과 병행 가능).
- **DNS (서브존)**: Route53은 외부 관리. 이름·MetalLB 매핑만 [`dns/inventory.yaml`](dns/inventory.yaml). 프록시 Host 분기: [`cli/ops/proxy.md`](cli/ops/proxy.md).

### 1차 데이터 노드 (raw k8s)

| 노드 라벨 | 네임스페이스 | 비고 |
| --- | --- | --- |
| mysql | mysql | TCP |
| redis | redis | TCP |
| kafka | kafka | TCP |
| rabbitmq | rabbitmq | UI `rabbitmq.nginx.lab.origemite.com` |
| vault | vault | UI `vault.nginx.lab.origemite.com` |

### 플랫폼·관측 (Helm)

| values | 역할 |
| --- | --- |
| `metallb.yaml` | LoadBalancer IP |
| `nginx.yaml` | Ingress 컨트롤러 |
| `cert-manager.yaml` | 인증서 |
| `kube-prometheus-stack.yaml` | Prometheus + Grafana |
| `loki.yaml` / `fluent-bit.yaml` | 로그 |
| `tempo.yaml` / `opentelemetry-collector.yaml` | 트레이스 (Zipkin 대신 Tempo) |
| `argocd.yaml` | GitOps |
| `headlamp.yaml` | 클러스터 UI |
| `opensearch.yaml` | 로그 검색 2차 (ES/Kibana 대신) |
| `cilium.yaml` | CNI (Phase 6A) |
| `istiod.yaml` / `kiali.yaml` | 서비스 메시·메시 UI (Phase 6B) |
| `harbor.yaml` | 컨테이너 레지스트리 (Phase 6C) |
| `keycloak.yaml` | IAM (Phase 6D) |
| `kong.yaml` / `traefik.yaml` / `haproxy-ingress.yaml` | Ingress 비교 (Phase 7) |
| `envoy-gateway.yaml` / `istio-gateway.yaml` | Gateway API 비교 (Phase 7) |

Phase 6에서 Cilium, Istio, Harbor, Keycloak을 이 랩에 추가한다. Phase 7에서 진입점 7종을 비교한다. **Kubeflow, Rook, Teleport**는 이 클러스터에 넣지 않는다.

## 저장소

```text
helm/values/     # 외부 차트 values
k8s/             # namespace node deployment service ingress gateway configmap
argocd/          # project / application (다음 단계)
terraform/       # 클러스터 부트스트랩 (다음 단계)
cli/doc/         # 기본 명령 (<자리표시자>)
cli/ops/         # 실명 복붙 명령
ref/             # 2026 표준 등 외부 참고
```

## CLI

- [`cli/doc/`](cli/doc/) — 자주 쓰는 기본 명령
- [`cli/ops/`](cli/ops/) — 클러스터 `lab` 기준 복붙 명령 (`k3d`, `helm`, `kube`, `metallb`, `cert-manager`, `cilium`, `istio`, `harbor`, `keycloak`, `ingress-compare`, …)

## 참고

- [`ref/Ref.md`](ref/Ref.md) — 2026년 쿠버네티스 표준 아키텍처 요약·링크
