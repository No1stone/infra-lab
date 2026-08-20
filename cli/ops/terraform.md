# terraform ops

> **Experimental / 최소 구현 예정** — `terraform/envs/lab/`는 `.gitkeep`만 있음. **당장은 [`k3d.md`](k3d.md)로 클러스터 생성.**

k3d `lab` 클러스터 부트스트랩 Terraform — **필수 커리큘럼(Phase 0T)**.

## 현재 (수동)

```bash
cat terraform/README.md
ls -la terraform/envs/lab/
```

클러스터 생성 — [`k3d.md`](k3d.md) Cilium·MetalLB용 create 블록.

## 계획 리소스 (`terraform/envs/lab`)

| # | 리소스 | 값 / 비고 |
| --- | --- | --- |
| 1 | cluster name | `lab` |
| 2 | servers | `1` |
| 3 | agents | `5` |
| 4 | LB ports | `80:80@loadbalancer`, `443:443@loadbalancer` |
| 5 | k3s disable | traefik, servicelb, network-policy (Cilium 경로) |
| 6 | node labels | `lab.origemite.com/workload=*` — `k8s/node/*/labels.yaml` |
| 7 | kubeconfig | merge + context `k3d-lab` |
| 8 | (범위 밖) | Helm·MetalLB·cert-manager — Terraform 미포함 |

구현 방식 후보: `null_resource` + `local-exec`로 `k3d cluster create …` 래핑. provider는 **k3d 공식 TF provider 검토 전** local/null만.

## 향후 init / apply (미구현)

```bash
terraform -chdir=terraform/envs/lab init
terraform -chdir=terraform/envs/lab plan
terraform -chdir=terraform/envs/lab apply
terraform -chdir=terraform/envs/lab destroy
```

destroy는 k3d 클러스터만 삭제 — **Route53·origemite.com DNS 무영향**.

## 관련

- [`terraform/README.md`](../../terraform/README.md)
- [`cli/doc/terraform.md`](../doc/terraform.md)
- [`doc/03-terraform.md`](../../doc/03-terraform.md)
