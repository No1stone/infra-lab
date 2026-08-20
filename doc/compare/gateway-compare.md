# 게이트웨이·Ingress 컨트롤러 비교

Phase 7에서 **7개 LB 데모**(`demo-echo`, MetalLB `.201`–`.207`)는 [`cli/ops/ingress-compare.md`](../../cli/ops/ingress-compare.md)에 있다. 이 문서는 **기능 매트릭스**와 **향후 확장 체크리스트**(HTTP/HTTPS/gRPC/WebSocket/Rewrite/Header/RateLimit/JWT)용이다.

대상: **NGINX Gateway Fabric**, **Envoy Gateway**, **Cilium Gateway**, **Kong**, **Traefik**, **Istio Gateway**. 랩 Phase 7에 포함된 **ingress-nginx**·**HAProxy Ingress**는 참고 열로 함께 적는다.

## 요약 매트릭스

| 항목 | NGINX Gateway Fabric | Envoy Gateway | Cilium Gateway | Kong | Traefik | Istio Gateway | ingress-nginx (랩) | HAProxy Ingress (랩) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **Gateway API** | ★ 네이티브 | ★ 네이티브 | ★ 네이티브 (Cilium 내장) | ◐ Kong Gateway + 플러그인 | ◐ v3 Gateway provider | ★ Gateway + VirtualService | ❌ Ingress v1 | ❌ Ingress v1 |
| **Ingress** | ❌ (Fabric는 Gateway API) | ◐ 레거시 호환 제한 | ❌ (values에서 Ingress Controller off) | ★ Ingress + Kong Ingress | ★ IngressRoute + Ingress | ◐ VirtualService/legacy | ★ Phase 1·7 | ★ Phase 7 |
| **gRPC** | ◐ GRPCRoute | ★ GRPCRoute | ◐ GRPCRoute (버전 의존) | ★ 플러그인·Ingress annotations | ◐ | ★ | ◐ annotations | ◐ |
| **mTLS** | ◐ Backend TLS·Gateway TLS | ◐ BackendTLSPolicy 등 | ◐ Gateway TLS + Cilium mesh(별도) | ★ upstream mTLS 플러그인 | ◐ | ★ mesh + Gateway TLS | ◐ auth-tls secret | ◐ |
| **Observability** | ◐ NGINX metrics | ★ Envoy access/metrics | ★ Hubble L7 + Prometheus | ★ 플러그인·Admin API | ◐ metrics | ★ Kiali·mesh telemetry | ◐ nginx metrics | ◐ HAProxy stats |
| **Network Policy** | ❌ (CNI/별도) | ❌ | ★ CiliumNetworkPolicy와 동일 CNI | ❌ | ❌ | ◐ AuthorizationPolicy | ❌ | ❌ |
| **ops 난이도** | 중 | 중 | 중 (CNI+Gateway 결합) | 중~높음 (DB·플러그인) | 낮음~중 | **높음** (control plane·sidecar) | **낮음** | 중 |

**랩 Phase 7 실습**: ingress-nginx, Envoy Gateway, Cilium Gateway, Kong, Traefik, Istio Gateway, HAProxy — HTTP Host 라우팅 ✅

**미포함·별도 검토**: NGINX Gateway Fabric — ingress-nginx EOL 후 Gateway API 대안. values/데모는 추후 추가 예정 📋

## 프로토콜·라우팅 확장 체크리스트

동일 `demo-echo` 또는 echo/grpc 데모 앱으로 **컨트롤러별** 채울 항목. 상세 스캐폴드: [`cli/ops/gateway-api-features.md`](../../cli/ops/gateway-api-features.md)

| 기능 | NGINX GF | Envoy GW | Cilium GW | Kong | Traefik | Istio GW | ingress-nginx | HAProxy |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| HTTP | 📋 | ✅ Phase 7 | ✅ Phase 7 | ✅ Phase 7 | ✅ Phase 7 | ✅ Phase 7 | ✅ Phase 7 | ✅ Phase 7 |
| HTTPS (TLS terminate) | 📋 | 📋 cert-manager | 📋 | 📋 | 📋 | 📋 | 📋 Phase 1 패턴 | 📋 |
| gRPC (GRPCRoute / reflection) | 📋 | 📋 | 📋 | 📋 | 📋 | 📋 | 📋 | 📋 |
| WebSocket | 📋 | 📋 | 📋 | 📋 | 📋 | 📋 | 📋 | 📋 |
| Path/host Rewrite | 📋 | 📋 HTTPRoute filter | 📋 | 📋 | 📋 | 📋 VirtualService | 📋 | 📋 |
| Request/Response Header | 📋 | 📋 | 📋 | 📋 | 📋 | 📋 | 📋 | 📋 |
| Rate limit | 📋 | 📋 BackendTrafficPolicy | 📋 | 📋 rate-limiting plugin | 📋 | 📋 Envoy filter | 📋 | 📋 |
| JWT / OIDC | 📋 | 📋 ext auth / SecurityPolicy | 📋 | 📋 | 📋 | 📋 RequestAuthentication | 📋 | 📋 |

## 컨트롤러별 메모

### NGINX Gateway Fabric

- F5/NGINX의 **Gateway API** 구현. ingress-nginx와 별 제품.
- Kubernetes Ingress 리소스 대신 `Gateway` + `HTTPRoute` 중심.
- 랩: 아직 Helm/데모 없음. Phase 7 Envoy·Cilium·Istio와 같은 Host 패턴(`demo.<gw>.lab…`)으로 추가 검토.

### Envoy Gateway

- CNCF 쪽 Gateway API **레퍼런스 구현**에 가깝다. Envoy data plane.
- `helm/values/envoy-gateway.yaml`, `k8s/gateway/demo-envoy.yaml` — `demo.gateway.lab.origemite.com`, MetalLB `.202`.
- 확장: `BackendTrafficPolicy`, `SecurityPolicy`, GRPCRoute — EG CRD 버전 확인 필요.

### Cilium Gateway

- CNI와 **동일 바이너리**에서 Gateway API 처리. `gatewayAPI.enabled: true`, Ingress Controller는 off.
- `demo.cilium.lab.origemite.com`, `.203`. L7 정책·Hubble과 연계 가능 ★.
- mesh mTLS(Cilium mutual auth)는 Gateway TLS와 별 트랙 — [`service-mesh-compare.md`](service-mesh-compare.md)

### Kong

- Ingress + **Kong Gateway** 플러그인 생태계. DB-less / DB 모드 선택.
- `demo.kong.lab.origemite.com`, `.204`. JWT·rate limit·upstream mTLS는 플러그인으로 강함.
- Gateway API는 Kong 3.x+ Gateway controller로 ◐.

### Traefik

- IngressRoute CRD + Ingress dual. 설정이 비교적 단순, ops **낮음~중**.
- `demo.traefik.lab.origemite.com`, `.205`. Gateway API provider는 Traefik v3.

### Istio Gateway

- **Gateway API** + 기존 `Gateway`/`VirtualService` 병행. sidecar mesh와 한 스택.
- `demo.istio.lab.origemite.com`, `.206`. east-west mTLS는 [`cli/ops/mtls.md`](../../cli/ops/mtls.md). 엣지 JWT는 RequestAuthentication + AuthorizationPolicy.

### ingress-nginx / HAProxy Ingress (랩 기준선)

- **Ingress v1** 학습용. 2026 이후 EOL 방향 → Gateway API 전환 참고용.
- 플랫폼 기본 입구: `*.nginx.lab.origemite.com` (`.201`).

## 선택 가이드 (초안)

| 목적 | 1순위 후보 | 이유 |
| --- | --- | --- |
| Gateway API 표준 학습 | Envoy Gateway, Cilium Gateway | CRD 순정·문서 풍부 |
| ingress-nginx 대체 | NGINX Gateway Fabric, Envoy Gateway | Gateway API 네이티브 |
| L7 정책 + 입구 일원화 | Cilium Gateway | CiliumNetworkPolicy·Hubble과 동일 스택 |
| API 게이트웨이(JWT·quota) | Kong | 플러그인 |
| 빠른 실험·낮은 ops | Traefik, ingress-nginx | 설정 단순 |
| mesh + 동일 vendor 엣지 | Istio Gateway | Kiali·mTLS와 통합 |

## 관련

- Phase 7 ops: [`cli/ops/ingress-compare.md`](../../cli/ops/ingress-compare.md)
- 기능 확장 스캐폴드: [`cli/ops/gateway-api-features.md`](../../cli/ops/gateway-api-features.md)
- DNS·MetalLB: [`dns/inventory.yaml`](../../dns/inventory.yaml)
- 튜토리얼: [`doc/14-ingress-compare.md`](../14-ingress-compare.md)
