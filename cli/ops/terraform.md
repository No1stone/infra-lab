# terraform ops

k3d `lab` 클러스터 부트스트랩 Terraform — **필수 커리큘럼(Phase 0T)**. `terraform/`는 현재 스텁, 구현 예정(`null`/`local` 또는 k3d CLI 래핑).

당장은 [`k3d.md`](k3d.md) 수동 생성. IaC 추가 시 아래 디렉터리에서 init/apply.

```bash
ls terraform/
cat terraform/README.md
```

향후 예시(미구현):

```bash
terraform -chdir=terraform init
terraform -chdir=terraform plan
terraform -chdir=terraform apply
```
