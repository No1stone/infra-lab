# infra-lab 참고 자료

`ref/` 폴더는 **infra-lab 홈랩 실습과 직접 관련된 외부 참고 링크**를 모아 둔 곳이다.  
이 파일(`Ref.md`)은 그중 핵심 자료의 요약·링크·랩 맥락 메모를 한곳에 정리한다.

---

## 2026년 쿠버네티스 표준 아키텍처

「쿠버네티스 인프라 구축·운영 노하우 3/e」(_Book_k8sInfra)의 **2026년판 표준 아키텍처** 문서.

| 구분 | 링크 |
| --- | --- |
| 폴더 | [docs/k8s-stnd-arch/2026](https://github.com/sysnet4admin/_Book_k8sInfra/tree/main/docs/k8s-stnd-arch/2026) |
| README (한국어) | [README.md](https://github.com/sysnet4admin/_Book_k8sInfra/blob/main/docs/k8s-stnd-arch/2026/README.md) |
| README (English) | [README_en.md](https://github.com/sysnet4admin/_Book_k8sInfra/blob/main/docs/k8s-stnd-arch/2026/README_en.md) |
| PDF | [2026-k8s-stnd-arch.pdf](https://github.com/sysnet4admin/_Book_k8sInfra/blob/main/docs/k8s-stnd-arch/2026/2026-k8s-stnd-arch.pdf) |

### 왜 보는가 (infra-lab 맥락)

- **CNCF Graduated·Landscape**를 기준으로 컴포넌트를 고르는 배경을 이해할 수 있다.
- **2026년 변화**(OCI 표준 채택, Ingress NGINX 지원 종료 등)를 미리 알고 랩 설계에 반영한다.
- infra-lab 실습 스택(**Terraform · Helm · Argo CD**)과 겹치는 **사실상(de facto) 표준** 조합을 확인한다.
- 현재·후보 워크로드(mysql, redis, kafka 등)를 **운영 표준 스택**과 대조해 다음 단계를 정한다.

### 문서 핵심 요약

- **선택 기준**: CNCF Graduated 프로젝트와 Landscape를 참고해 검증된 오픈소스 위주로 구성.
- **2026 주요 변화**
  - **OCI** 채택 확대
  - **Ingress NGINX** 공식 지원 종료 — 2026년 3월 이후 EOL. 프로덕션에서는 **Gateway API** 전환 권장. 다만 현장에서는 여전히 널리 쓰이므로, 랩에서는 학습·실습 목적으로 유지하되 **마이그레이션 경로**를 염두에 둔다.
- **권장 스택 예시**: Teleport, Keycloak, HAProxy, Calico/Cilium, MetalLB, **Helm**, Headlamp(Lens 대체), Istio, Docker, GitHub Actions, **Argo CD**, Harbor, **Prometheus**, **Grafana**, OpenSearch(로그·ES/Kibana 대체), Tempo(트레이스), **Vault**, cert-manager 등.
- **배포·GitOps**: **Helm + Argo CD**가 사실상 표준.

### infra-lab 관련 컴포넌트 대조

| 컴포넌트 | infra-lab | 표준 문서 쪽 메모 |
| --- | --- | --- |
| Ingress | **ingress-nginx** (`nginx` NS, `*.nginx.lab.origemite.com`) | EOL 예정 → Gateway API 전환 검토 |
| GitOps | **Argo CD** (실습) | 사실상 표준 |
| 패키징 | **Helm** (실습) | 사실상 표준 |
| IaC | **Terraform** (실습) | 클러스터·부트스트랩 |
| mysql / redis / kafka / rabbitmq | **1차 워크로드** | 앱·데이터 계층 |
| vault | **1차 워크로드** | 문서 권장과 일치 |
| prometheus / grafana | **후보** | 모니터링 표준 |
| elasticsearch / kibana | **후보** | 로그는 OpenSearch 쪽 권장 |
| fluentbit / otel / loki / zipkin | **후보** | 관측·로그·트레이스 |
| Istio / Harbor / Keycloak / MetalLB 등 | 미도입 | 프로덕션 참고용 |

> **랩 환경**: AWS 프록시 → reverse SSH → Ubuntu 노트북 **k3d**, 호스트 3뎁스 `<앱>.<게이트웨이>.lab.origemite.com` (2뎁스 금지).

---

## 연도별 아카이브

같은 시리즈의 이전·이후 연도판:

| 연도 | 폴더 |
| --- | --- |
| 2022 | [docs/k8s-stnd-arch/2022](https://github.com/sysnet4admin/_Book_k8sInfra/tree/main/docs/k8s-stnd-arch/2022) |
| 2023 | [docs/k8s-stnd-arch/2023](https://github.com/sysnet4admin/_Book_k8sInfra/tree/main/docs/k8s-stnd-arch/2023) |
| 2024 | [docs/k8s-stnd-arch/2024](https://github.com/sysnet4admin/_Book_k8sInfra/tree/main/docs/k8s-stnd-arch/2024) |
| 2025 | [docs/k8s-stnd-arch/2025](https://github.com/sysnet4admin/_Book_k8sInfra/tree/main/docs/k8s-stnd-arch/2025) |
| 2026 | [docs/k8s-stnd-arch/2026](https://github.com/sysnet4admin/_Book_k8sInfra/tree/main/docs/k8s-stnd-arch/2026) |

---

## 저작·크레딧

「쿠버네티스 인프라 구축·운영 노하우 3/e」(_Book_k8sInfra) — 아래 저자·기여자의 자료를 참고한다.

| 이름 | GitHub |
| --- | --- |
| 조훈 | [sysnet4admin](https://github.com/sysnet4admin) |
| 심근우 | [gnu-gnu](https://github.com/gnu-gnu) |
| 문성주 | [seongjumoon](https://github.com/seongjumoon) |
| 이성민 | [sungmincs](https://github.com/sungmincs) |


