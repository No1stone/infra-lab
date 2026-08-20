# 운영 시나리오

프로덕션에 가까운 **운영, 복구, 배포** 절차를 랩(k3d `lab`)에서 단계별로 연습한다. 본문은 서술, 맥락, **복붙 명령**은 [`cli/ops/`](../../cli/ops/)를 따른다.

## 목차

| # | 시나리오 | 핵심 기술 |
| --- | --- | --- |
| 01 | [무중단 롤링 업데이트](01-zero-downtime-update.md) | Deployment strategy, readiness, PDB |
| 02 | [노드 업그레이드](02-node-upgrade.md) | cordon, drain, PDB |
| 03 | [블루-그린 배포](03-blue-green.md) | Service selector, Ingress 전환 |
| 04 | [카나리 배포](04-canary.md) | nginx canary, Istio, **Argo Rollouts** |
| 05 | [재해 복구](05-disaster-recovery.md) | 백업, 클러스터 재생성, GitOps |
| 06 | [클러스터 업그레이드](06-cluster-upgrade.md) | k3s/k3d, Helm, CRD 순서 |
| 07 | [네트워크 장애](07-network-failure.md) | Cilium, NetworkPolicy, mTLS |
| 08 | [인증서 갱신](08-certificate-renewal.md) | cert-manager, Ingress TLS |
| 09 | [Secret 로테이션](09-secret-rotation.md) | Secret, reload, External Secrets(계획) |
| 10 | [게이트웨이 마이그레이션](10-gateway-migration.md) | ingress-compare, DNS, MetalLB |

## 사전 공통

- 클러스터: [`doc/02-k3d-cluster.md`](../02-k3d-cluster.md), [`cli/ops/k3d.md`](../../cli/ops/k3d.md)
- 관측: [`doc/06-observability.md`](../06-observability.md), [`cli/ops/otel-trace-lab.md`](../../cli/ops/otel-trace-lab.md)
- 카오스: [`cli/ops/chaos.md`](../../cli/ops/chaos.md)

## 튜토리얼과의 관계

| 경로 | 역할 |
| --- | --- |
| `doc/0x-*.md` | Phase별 **입문, 설치** 튜토리얼 |
| `doc/scenarios/` (여기) | **운영 시나리오** — 여러 Phase를 엮은 end-to-end |
| `cli/ops/` | 실명 복붙 명령 |
