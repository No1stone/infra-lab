# infra-lab 참고 자료

`ref/` — 홈랩 실습과 맞닿은 **외부 참고**와, 이 저장소 기준 **현재 스택 대조**를 둔다.  
실습 절차·복붙 명령은 [`doc/`](../doc/README.md)·[`cli/ops/`](../cli/ops/)를 본다.

---

## 랩 한줄 요약 (2026-08 기준)

| 항목 | 내용 |
| --- | --- |
| 런타임 | Ubuntu 노트북 **k3d** `lab` (Mac에서 클러스터 기동 안 함) |
| 공개 입구 | AWS 프록시 → reverse SSH → MetalLB `.201`–`.207` |
| DNS | **3뎁스만** `<앱>.<게이트웨이>.lab.origemite.com` (2뎁스 금지) |
| Route53 | `origemite.com` **개인용·외부 관리**. 이 저장소/TF로 변경·destroy 안 함. 이름만 [`dns/inventory.yaml`](../dns/inventory.yaml) |
| 필수 실습 | Terraform(스텁→구현), Keycloak, Istio mTLS, Cilium NetworkPolicy |
| 제외 | Kubeflow(별도 머신), Rook, Teleport, Route53/Gitea/Jenkins를 TF로 관리 |

---

## 2026년 쿠버네티스 표준 아키텍처

「쿠버네티스 인프라 구축·운영 노하우 3/e」(_Book_k8sInfra) **2026년판**.

| 구분 | 링크 |
| --- | --- |
| 폴더 | [docs/k8s-stnd-arch/2026](https://github.com/sysnet4admin/_Book_k8sInfra/tree/main/docs/k8s-stnd-arch/2026) |
| README (한국어) | [README.md](https://github.com/sysnet4admin/_Book_k8sInfra/blob/main/docs/k8s-stnd-arch/2026/README.md) |
| README (English) | [README_en.md](https://github.com/sysnet4admin/_Book_k8sInfra/blob/main/docs/k8s-stnd-arch/2026/README_en.md) |
| PDF | [2026-k8s-stnd-arch.pdf](https://github.com/sysnet4admin/_Book_k8sInfra/blob/main/docs/k8s-stnd-arch/2026/2026-k8s-stnd-arch.pdf) |

### 왜 보는가

- CNCF Graduated·Landscape 기준으로 컴포넌트를 고르는 배경
- **2026**: OCI 확대, **Ingress NGINX EOL** → Gateway API 전환 압력
- 랩 스택(Terraform · Helm · Argo CD · Cilium · Istio · Keycloak · Harbor)과 de facto 표준 대조

### 문서 핵심 요약

- 검증된 오픈소스(CNCF) 위주
- Ingress NGINX: 2026-03 이후 EOL 방향 → 프로덕션은 Gateway API 권장. 랩은 **학습용 nginx 유지 + Phase 7에서 Gateway/타 컨트롤러 비교**
- 권장 예: Cilium/Calico, MetalLB, Helm, Headlamp, Istio, Argo CD, Harbor, Keycloak, Prometheus/Grafana, OpenSearch, Tempo, Vault, cert-manager, (문서 예) Teleport·HAProxy·GitHub Actions
- 배포: **Helm + Argo CD**

---

## infra-lab ↔ 표준 문서 대조

| 컴포넌트 | infra-lab 상태 | 표준·메모 |
| --- | --- | --- |
| IaC | **Terraform** Phase 0T (스텁→`envs/` 분리 예정). Route53 제외 | 클러스터 부트스트랩 |
| CNI | **Cilium** + Hubble + **CiliumNetworkPolicy** (파드간 권한) | Calico/Cilium |
| 외부 LB | **MetalLB** `.201`–`.207` (입구 IP ≈ 클라우드 NLB 자리) | MetalLB |
| 플랫폼 Ingress | **ingress-nginx** (기본). 호스트 `*.nginx.lab…` | EOL → Gateway API |
| Gateway API | Phase 7: **Envoy / Cilium / Istio** Gateway | 2026 전환 축 |
| Ingress 비교 | Kong, Traefik, HAProxy (+ nginx) | 학습·비교 |
| GitOps | **Argo CD** (+ Application은 이후) | de facto |
| 패키징 | **Helm** values under `helm/values/` | de facto |
| IAM | **Keycloak** (필수) | 문서 권장 |
| 메시·mTLS | **Istio** + Kiali + PeerAuthentication STRICT | 문서 권장 |
| 레지스트리 | **Harbor** (≈ ECR 역할) | 문서 권장 |
| 관측 | kube-prometheus-stack, Loki, Fluent Bit, **Tempo**, OTel | Zipkin→Tempo, ES→OpenSearch |
| 로그 검색 | OpenSearch (선택 Phase 5) | ES/Kibana 대체 |
| UI | **Headlamp** | Lens 대체 |
| 시크릿·데이터 | Vault, mysql, redis, kafka, rabbitmq | 1차 워크로드 |
| CI | 계획: Gitea / Jenkins·GHA (개인 인프라, TF 밖) | GH Actions 문서 예 |
| 점진 배포 | 계획: Argo Rollouts | — |
| Teleport / Rook / Kubeflow | **이 클러스터 제외** | 문서엔 있으나 랩 RAM·범위 밖 |

---

## 진입·DNS·라우팅 (랩 규칙)

```text
DNS 3뎁스  →  MetalLB(게이트웨이별 IP)  →  Ingress|Gateway 리소스(Host)  →  Service DNS → Pod
```

- 한 요청은 **고른 게이트웨이 하나**만 탐 (7개 동시 팬아웃 아님)
- Phase 7: 같은 `demo-echo`를 Host만 바꿔 7입구로 비교 — [`cli/ops/ingress-compare.md`](../cli/ops/ingress-compare.md)
- 클러스터 내부: `http://서비스.네임스페이스` (CoreDNS). 외부 진입용이 아님
- 공식: [Gateway API](https://gateway-api.sigs.k8s.io/), [MetalLB](https://metallb.universe.tf/)

---

## 컴포넌트별 공식 문서 (짧게)

| 주제 | 링크 | 랩 경로 |
| --- | --- | --- |
| Cilium / NetworkPolicy | [Cilium Network Policy](https://docs.cilium.io/en/stable/security/policy/) | [`k8s/ciliumnetworkpolicy/`](../k8s/ciliumnetworkpolicy/), [`cli/ops/cilium.md`](../cli/ops/cilium.md) |
| Hubble | [Hubble](https://docs.cilium.io/en/stable/observability/hubble/) | values `hubble.enabled` |
| Istio mTLS | [Mutual TLS](https://istio.io/latest/docs/tasks/security/authentication/mtls-migration/) | [`cli/ops/mtls.md`](../cli/ops/mtls.md) |
| PeerAuthentication | [AuthN policy](https://istio.io/latest/docs/reference/config/security/peer_authentication/) | [`k8s/peerauthentication/`](../k8s/peerauthentication/) |
| Argo CD | [argo-cd.readthedocs.io](https://argo-cd.readthedocs.io/) | `helm/values/argocd.yaml` |
| Harbor | [goharbor.io/docs](https://goharbor.io/docs/) | `helm/values/harbor.yaml` |
| Keycloak | [keycloak.org/guides](https://www.keycloak.org/guides) | `helm/values/keycloak.yaml` |
| cert-manager | [cert-manager.io](https://cert-manager.io/docs/) | Phase 1 |
| OpenTelemetry | [opentelemetry.io](https://opentelemetry.io/docs/) | OTel Collector values |

---

## 연도별 아카이브 (_Book_k8sInfra)

| 연도 | 폴더 |
| --- | --- |
| 2022 | [2022](https://github.com/sysnet4admin/_Book_k8sInfra/tree/main/docs/k8s-stnd-arch/2022) |
| 2023 | [2023](https://github.com/sysnet4admin/_Book_k8sInfra/tree/main/docs/k8s-stnd-arch/2023) |
| 2024 | [2024](https://github.com/sysnet4admin/_Book_k8sInfra/tree/main/docs/k8s-stnd-arch/2024) |
| 2025 | [2025](https://github.com/sysnet4admin/_Book_k8sInfra/tree/main/docs/k8s-stnd-arch/2025) |
| 2026 | [2026](https://github.com/sysnet4admin/_Book_k8sInfra/tree/main/docs/k8s-stnd-arch/2026) |

---

## 저작·크레딧

「쿠버네티스 인프라 구축·운영 노하우 3/e」(_Book_k8sInfra)

| 이름 | GitHub |
| --- | --- |
| 조훈 | [sysnet4admin](https://github.com/sysnet4admin) |
| 심근우 | [gnu-gnu](https://github.com/gnu-gnu) |
| 문성주 | [seongjumoon](https://github.com/seongjumoon) |
| 이성민 | [sungmincs](https://github.com/sungmincs) |
