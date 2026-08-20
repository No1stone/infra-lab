# 서비스 메시 비교 — Istio, Cilium mesh, Linkerd

east-west **mTLS·트래픽 제어·관측** 관점 비교. infra-lab 랩은 **Istio + PeerAuthentication STRICT**만 full 실습(Phase 6B-m). Cilium mesh·Linkerd는 **계획/개념** 📋.

## 요약 매트릭스

| 항목 | Istio | Cilium (mesh) | Linkerd |
| --- | --- | --- | --- |
| **mTLS** | ★ PeerAuthentication / auto MTLS | ★ Mutual auth (SPIRE 등) | ★ 기본 mTLS on |
| **Retry** | ★ VirtualService | ◐ HTTP retry policy | ★ ServiceProfile |
| **Circuit breaker** | ★ DestinationRule | ◐ | ★ |
| **Traffic split** | ★ VirtualService weight | ◐ Gateway/Ingress 분리 | ★ TrafficSplit |
| **Observability** | ★ Kiali, mesh metrics | ★ Hubble (L3/L7) | ★ viz, metrics |
| **ops 난이도** | **높음** | 중 (CNI 이미 있을 때) | **낮음~중** |
| **랩 상태** | ✅ [`cli/ops/mtls.md`](../../cli/ops/mtls.md) | 📋 개념 | 📋 개념 |

## 기능 상세

| 기능 | Istio | Cilium mesh | Linkerd |
| --- | --- | --- | --- |
| Sidecar | ✅ envoy sidecar ( 또는 ambient ) | ❌ sidecarless 지향 | ✅ ultra-light proxy |
| North-south | Gateway / Gateway API | Cilium Gateway + mesh | ◐ |
| L7 authz | AuthorizationPolicy | CiliumNetworkPolicy L7 | ServerAuthorization |
| Multi-cluster | ◐ | ◐ | ◐ |
| 리소스 overhead | 높음 (sidecar per pod) | 낮음~중 (CNI 통합) | 낮음 |

## mTLS 모델 비교 — Istio STRICT vs Cilium Mutual Auth vs Envoy/Kong upstream mTLS

랩에서 **지금 full 실습 가능한 것은 Istio east-west STRICT** 뿐이다. 나머지는 개념·향후 Phase.

### Istio PeerAuthentication STRICT (✅ 랩)

| 항목 | 내용 |
| --- | --- |
| 범위 | **Pod ↔ Pod** (data plane proxy 간) |
| 신원 | SPIFFE ID (istio trust domain) |
| 설정 | `PeerAuthentication` `mode: STRICT` — [`k8s/peerauthentication/istio-mtls.yaml`](../../k8s/peerauthentication/istio-mtls.yaml) |
| 검증 | `istioctl authn tls-check`, Kiali lock icon |
| 엣지 TLS | **별개** — Ingress/Gateway HTTPS는 cert-manager·Gateway TLS |

```text
[Client Pod + sidecar] -- mTLS --> [Server Pod + sidecar]
         ↑ PeerAuthentication STRICT
```

### Cilium Mutual Auth (📋 계획/개념)

| 항목 | 내용 |
| --- | --- |
| 범위 | Cilium **identity** 기반 Pod 간 암호화 |
| 신원 | Cilium security identity / SPIFFE (버전·설정 의존) |
| sidecar | **없음** — eBPF/Cilium agent에서 처리 |
| Gateway | Cilium Gateway TLS terminate ≠ mesh mutual auth (레이어 분리) |
| 랩 | Istio와 **동시 full mesh 비권장**. CNI=Cilium 유지 시 mesh 트랙 **택일** 검토 |

```text
[Pod A] ---- Cilium encrypted tunnel ---- [Pod B]
              (mutual auth, no sidecar)
```

### Envoy / Kong upstream mTLS (📋 계획/개념 — north-south·egress)

| 항목 | Envoy Gateway | Kong |
| --- | --- | --- |
| 범위 | **클라이언트↔Gateway** 또는 **Gateway↔upstream** TLS | Gateway↔upstream, client cert |
| mesh | ❌ Pod 간 mTLS 아님 | ❌ |
| 설정 | `BackendTLSPolicy`, Secret 참조 | `mtls-auth`, upstream certificate |
| 랩 | Phase 7 HTTP만. HTTPS upstream 📋 | Phase 7 HTTP만. JWT·mtls plugin 📋 |

```text
Internet --TLS--> [Envoy/Kong Gateway] --TLS(mTLS)--> [Backend Service]
                      ↑ upstream mTLS (not mesh)
```

### 한눈에 구분

| 구분 | Istio STRICT | Cilium mutual auth | Envoy/Kong upstream mTLS |
| --- | --- | --- | --- |
| 레이어 | L7 proxy 간 (L4 tunnel) | CNI identity encryption | Gateway → backend |
| Sidecar | 필요 (classic) | 불필요 | Gateway Pod만 |
| 랩 실습 | ✅ | 📋 | 📋 |
| 문서 | [`cli/ops/mtls.md`](../../cli/ops/mtls.md) | Cilium docs mesh | [`gateway-compare.md`](gateway-compare.md) |

## Retry · Circuit breaker · Traffic split

| 패턴 | Istio (✅/📋) | Cilium (📋) | Linkerd (📋) |
| --- | --- | --- | --- |
| Retry | `VirtualService.retries` | HTTP policy / proxy 설정 | `ServiceProfile` |
| Circuit breaker | `DestinationRule` `connectionPool` / outlierDetection | 제한적 | `ServiceProfile` |
| Canary / split | `VirtualService` weight | Gateway/Ingress 분리 또는 mesh policy | `TrafficSplit` CRD |

랩 다음 단계: Istio `VirtualService` retry·split을 `ingress-compare` NS에 📋 추가.

## ops 난이도 (홈랩 기준)

| 메시 | 설치·업그레이드 | 디버깅 | 메모리 |
| --- | --- | --- | --- |
| Linkerd | 낮음 | 중 (`linkerd viz`) | 낮음 ★ |
| Cilium mesh | 중 (CNI已有) | Hubble ★ | 중 |
| Istio | **높음** | Kiali·istioctl ★ | **높음** (sidecar) |

infra-lab는 **Istio + Kiali + STRICT mTLS**를 필수 커리큘럼으로 채택 — ops 비용을 실습 비용으로 받아들임.

## 선택 가이드 (초안)

| 목적 | 추천 |
| --- | --- |
| L7 policy·트래픽·업계 표준 학습 | **Istio** (랩 선택) |
| Sidecar 없이 encryption | Cilium mesh |
| 최소 footprint·빠른 도입 | Linkerd |
| API GW upstream TLS only | Envoy Gateway / Kong (mesh 아님) |

## 관련

- mTLS ops: [`cli/ops/mtls.md`](../../cli/ops/mtls.md)
- Istio ops: [`cli/ops/istio.md`](../../cli/ops/istio.md)
- CNI·Hubble: [`cni-compare.md`](cni-compare.md)
- Gateway·JWT: [`gateway-compare.md`](gateway-compare.md)
