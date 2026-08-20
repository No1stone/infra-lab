# 비교 문서 (compare)

infra-lab에서 **CNI·게이트웨이·서비스 메시** 후보를 한곳에 모아 둔 **기능 매트릭스**입니다. Phase 7 LB 데모([`cli/ops/ingress-compare.md`](../../cli/ops/ingress-compare.md))와 별개로, 선택·확장 시 참고용 초안입니다.

## 문서 목록

| 문서 | 대상 | 랩 상태 |
| --- | --- | --- |
| [gateway-compare.md](gateway-compare.md) | NGINX Gateway Fabric, Envoy Gateway, Cilium Gateway, Kong, Traefik, Istio (+ ingress-nginx·HAProxy) | Phase 7 HTTP LB 데모 완료. HTTPS/gRPC 등은 체크리스트 |
| [cni-compare.md](cni-compare.md) | Calico, Cilium, Flannel | **Cilium** 실습(Phase 6A). Calico·Flannel은 개념 |
| [service-mesh-compare.md](service-mesh-compare.md) | Istio, Cilium mesh, Linkerd | **Istio mTLS** 실습(Phase 6B). 나머지는 계획/개념 |

## 이 폴더와 다른 경로

| 경로 | 역할 |
| --- | --- |
| **`doc/compare/`** (여기) | 표·체크리스트 중심 **비교·선택** |
| [`doc/`](../README.md) | 단계별 튜토리얼 서술 |
| [`cli/ops/`](../../cli/ops/) | 클러스터 `lab` 기준 복붙 명령 |
| [`cli/ops/gateway-api-features.md`](../../cli/ops/gateway-api-features.md) | Phase 7 → 고급 Gateway 기능 확장 스캐폴드 |

## 범례

| 표기 | 의미 |
| --- | --- |
| ✅ | 랩에서 이미 실습·검증 |
| ◐ | 부분 지원 또는 values/매니페스트만 준비 |
| 📋 | 계획·체크리스트 (미실습) |
| ❌ | 미지원 또는 이 랩 범위 밖 |
| ★ | 해당 영역에서 강점 |

운영 난이도: **낮음** < **중** < **높음** (학습·홈랩 기준, 프로덕션 SLA는 별도).

## 관련

- Phase 7 진입점: [`cli/ops/ingress-compare.md`](../../cli/ops/ingress-compare.md)
- Cilium NetworkPolicy: [`cli/ops/cilium.md`](../../cli/ops/cilium.md)
- Istio mTLS: [`cli/ops/mtls.md`](../../cli/ops/mtls.md)
- 2026 표준 대조: [`ref/Ref.md`](../../ref/Ref.md)
