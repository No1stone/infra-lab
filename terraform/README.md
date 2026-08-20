# terraform

Phase 0T — k3d `lab` 클러스터 부트스트랩 IaC(서버, 에이전트, 포트, 노드 라벨). **필수 커리큘럼**, 구현은 **Experimental / 최소 구현 예정**.

**DNS / Route53 범위 밖.** `origemite.com`은 개인용 — destroy, apply가 실제 DNS에 영향 주면 안 된다. `aws_route53_*` 두지 않는다. 이름 목록만 [`dns/inventory.yaml`](../dns/inventory.yaml). 랩 전용 도메인은 나중에 구매, 연결.

## 상태

| 항목 | 상태 |
| --- | --- |
| `terraform/envs/lab/` | 디렉터리만 (`.gitkeep`) — **코드 없음** |
| 당장 클러스터 생성 | [`cli/ops/k3d.md`](../cli/ops/k3d.md) 수동 |
| IaC ops | [`cli/ops/terraform.md`](../cli/ops/terraform.md) |

## Experimental — 계획 리소스 (`envs/lab`)

다음을 **주석, 로컬/null provider** 수준으로 옮길 예정. fake provider로 `terraform validate`가 깨지지 않게 한다.

| 리소스 | 내용 |
| --- | --- |
| k3d cluster | 이름 `lab` |
| topology | `--servers 1`, `--agents 5` |
| ports | `80:80@loadbalancer`, `443:443@loadbalancer` |
| k3s args | traefik off, servicelb off (MetalLB), network-policy off (Cilium) |
| node labels | `lab.origemite.com/workload=<mysql\|redis\|kafka\|…>` — `k8s/node/*/labels.yaml`와 동기 |
| kubeconfig | merge + context switch (local-exec 또는 output) |
| MetalLB pool / ClusterIssuer | **Terraform 밖** — `k8s/configmap/` + Helm (범위 분리) |

디렉터리 레이아웃 (목표):

```text
terraform/
 README.md
 envs/
 lab/
 .gitkeep ← 현재
 main.tf ← (예정) null_resource + local-exec k3d
 variables.tf
 outputs.tf
```

## 사용 (구현 후)

```bash
terraform -chdir=terraform/envs/lab init
terraform -chdir=terraform/envs/lab plan
terraform -chdir=terraform/envs/lab apply
terraform -chdir=terraform/envs/lab destroy # k3d cluster delete lab — DNS 무영향
```

## 관련

- 튜토리얼: [`doc/03-terraform.md`](../doc/03-terraform.md)
- 기본 명령: [`cli/doc/terraform.md`](../cli/doc/terraform.md)
