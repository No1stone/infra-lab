# 튜토리얼

infra-lab 실습을 단계별로 따라 하는 **서술형 가이드**입니다.

## 이 폴더와 다른 경로

| 경로 | 역할 |
| --- | --- |
| **`doc/`** (여기) | 튜토리얼 본문·캡처 참조. **서술·맥락** |
| [`cli/doc/`](../cli/doc/) | 자주 쓰는 기본 명령 (`<자리표시자>`) |
| [`cli/ops/`](../cli/ops/) | 클러스터 `lab` 기준 **복붙 실행** 명령 |
| [`doc/images/`](images/) | 챕터별 스크린샷 (`doc/images/<phase>/`) |
| [`doc/troubleshooting/`](troubleshooting/) | 공통·반복 에러 모음 |

- **캡처**: 각 챕터의 `images/<phase>/`에 PNG 등을 저장하고 본문에서 상대 경로로 참조한다.
- **에러**: 챕터 표에 요약, 상세는 `troubleshooting/`에 추가한다.
- **DNS**: `origemite.com` 개인 존은 변경하지 않는다. 랩 호스트는 **3뎁스만** (`<앱>.<게이트웨이>.lab…`). 2뎁스 금지. [`dns/inventory.yaml`](../dns/inventory.yaml)

전체 트리: [`tree.md`](tree.md)

## 목차

| # | 챕터 | 상태 |
| --- | --- | --- |
| 00 | [개요](00-overview.md) | 초안 |
| 01 | [프록시와 DNS](01-proxy-and-dns.md) | 초안 |
| 02 | [k3d 클러스터](02-k3d-cluster.md) | 초안 |
| 03 | [Terraform](03-terraform.md) | 초안 |
| 04 | [MetalLB·Ingress·인증서](04-metallb-ingress-cert.md) | 초안 |
| 05 | [데이터 워크로드](05-data-workloads.md) | 초안 |
| 06 | [관측](06-observability.md) | 초안 |
| 07 | [Argo CD·Headlamp](07-argocd-headlamp.md) | 초안 |
| 08 | [OpenSearch (선택)](08-opensearch.md) | 초안 |
| 09 | [Cilium](09-cilium.md) | 초안 |
| 10 | [Istio·Kiali](10-istio-kiali.md) | 초안 |
| 11 | [mTLS](11-mtls.md) | 초안 |
| 12 | [Harbor](12-harbor.md) | 초안 |
| 13 | [Keycloak](13-keycloak.md) | 초안 |
| 14 | [Ingress 비교](14-ingress-compare.md) | 초안 |
| 15 | [GitOps·OIDC](15-gitops-oidc.md) | 초안 |
| 90 | [파괴·리셋](90-destroy-and-reset.md) | 초안 |

## 트러블슈팅

- [트러블슈팅 가이드](troubleshooting/README.md)
- [이슈 템플릿](troubleshooting/_template.md)
