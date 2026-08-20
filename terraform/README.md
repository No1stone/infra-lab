# terraform

Phase 0T — k3d `lab` 클러스터 부트스트랩 IaC(서버·에이전트·포트). **필수 커리큘럼**, 구현은 다음 단계.

**DNS / Route53 범위 밖.** `origemite.com`은 개인용 — destroy·apply가 실제 DNS에 영향 주면 안 된다. `aws_route53_*` 두지 않는다. 이름 목록만 [`dns/inventory.yaml`](../dns/inventory.yaml). 랩 전용 도메인은 나중에 구매·연결.

현재 스텁. 당장은 [`cli/ops/k3d.md`](../cli/ops/k3d.md) 수동 생성.
