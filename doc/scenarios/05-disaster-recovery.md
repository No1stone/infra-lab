# 05 재해 복구

## 목표

**control plane·클러스터 전체 상실** 후 Git 저장소 매니페스트·Helm values·(선택) Terraform으로 랩을 재구성하고, 데이터 워크로드는 **백업 유무에 따른 RPO/RTO** 차이를 이해한다.

## 사전조건

- infra-lab Git clone (Mac/Ubuntu 노트북)
- DNS·프록시는 **외부 고정** — destroy해도 Route53 변경 없음 — [`dns/inventory.yaml`](../../dns/inventory.yaml)
- (선택) Harbor·Velero 백업 — [`doc/12-harbor.md`](../12-harbor.md)

## 절차

1. **현재 상태 기록** — `kubectl get all -A`, Helm release list
   ```bash
   helm list -A
   kubectl get ingress -A
   ```
2. **(DR 시뮬) 클러스터 삭제**
   ```bash
   k3d cluster delete lab
   ```
3. **재생성** — [`cli/ops/k3d.md`](../../cli/ops/k3d.md) (Cilium·MetalLB 플래그 포함)
4. **플랫폼 부트스트랩** — [`cli/ops/helm.md`](../../cli/ops/helm.md) Phase 1→2→3 순
5. **MetalLB·Issuer** — [`cli/ops/metallb.md`](../../cli/ops/metallb.md), [`cli/ops/cert-manager.md`](../../cli/ops/cert-manager.md)
6. **GitOps(선택)** — Argo CD Application 재적용 — [`cli/ops/argocd.md`](../../cli/ops/argocd.md)
7. **데이터 워크로드** — `redis`, `kafka`, `mysql` 등 `k8s/deployment/` 재apply — **empty volume = 데이터 유실** 강조
8. **(향후) Terraform** — `terraform/envs/lab` apply로 3–4 단계 대체 — [`cli/ops/terraform.md`](../../cli/ops/terraform.md)

## 검증

- 플랫폼 UI: `grafana.nginx.lab.origemite.com`, `argocd.nginx.lab.origemite.com`
- `demo.nginx.lab.origemite.com` echo 응답
- Prometheus targets UP
- (백업 복원 시) Redis `KEYS` / Kafka topic 존재

## 관련

- [`doc/90-destroy-and-reset.md`](../90-destroy-and-reset.md)
- [`cli/ops/k3d.md`](../../cli/ops/k3d.md)
- [`cli/ops/terraform.md`](../../cli/ops/terraform.md)
- [`doc/scenarios/06-cluster-upgrade.md`](06-cluster-upgrade.md)
