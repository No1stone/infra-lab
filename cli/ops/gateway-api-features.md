# gateway-api-features ops

Phase 7는 **HTTP Host 라우팅**만 검증한다 — [`ingress-compare.md`](ingress-compare.md). 이 파일은 **HTTPS / gRPC / WebSocket / Rewrite / Header / RateLimit / JWT** 를 컨트롤러별로 확장할 때 쓰는 **체크리스트 스캐폴드**다. 전부 구현하지 않는다.

기능 매트릭스: [`doc/compare/gateway-compare.md`](../../doc/compare/gateway-compare.md)

## 공통 사전조건

```bash
# Gateway API CRD (Phase 7와 동일)
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.1/standard-install.yaml

# cert-manager + ClusterIssuer (HTTPS 항목 공통)
kubectl get clusterissuer -A
```

데모 앱: `ingress-compare` NS의 `demo-echo`. gRPC/WebSocket은 별도 echo/grpc 이미지 또는 agnhost 확장 📋.

## 컨트롤러별 확장 체크리스트

| 기능 | Envoy GW | Cilium GW | Istio GW | Kong | Traefik | ingress-nginx | HAProxy |
| --- | --- | --- | --- | --- | --- | --- | --- |
| HTTPS | 📋 Gateway TLS + cert-manager | 📋 | 📋 Gateway + cred | 📋 | 📋 | 📋 | 📋 |
| gRPC | 📋 GRPCRoute | 📋 | 📋 | 📋 | 📋 | 📋 | 📋 |
| WebSocket | 📋 HTTPRoute (upgrade) | 📋 | 📋 | 📋 | 📋 | 📋 | 📋 |
| Rewrite | 📋 URLRewrite filter | 📋 | 📋 VirtualService | 📋 | 📋 | 📋 | 📋 |
| Header | 📋 RequestHeaderModifier | 📋 | 📋 | 📋 | 📋 | 📋 | 📋 |
| RateLimit | 📋 BackendTrafficPolicy | 📋 | 📋 EnvoyFilter | 📋 plugin | 📋 | 📋 | 📋 |
| JWT | 📋 SecurityPolicy / ext auth | 📋 | 📋 RequestAuthentication | 📋 | 📋 | 📋 | 📋 |

✅ = Phase 7 HTTP 완료. 📋 = 아래 섹션 참고해 매니페스트 추가.

---

### Envoy Gateway (`demo.gateway.lab…`, `.202`)

| # | 항목 | 리소스·메모 | 상태 |
| --- | --- | --- | --- |
| 1 | HTTPS | `Gateway` `tls.certificateRefs`, HTTPRoute `parentRefs` | 📋 |
| 2 | gRPC | `GRPCRoute` → `demo-echo` grpc port | 📋 |
| 3 | WebSocket | HTTPRoute backend, timeout/upgrade | 📋 |
| 4 | Rewrite | HTTPRoute `filters.urlRewrite` | 📋 |
| 5 | Header | `RequestHeaderModifier` / `ResponseHeaderModifier` | 📋 |
| 6 | RateLimit | `BackendTrafficPolicy` (EG 버전 확인) | 📋 |
| 7 | JWT | `SecurityPolicy` 또는 ext auth gRPC | 📋 |

values: `helm/values/envoy-gateway.yaml` · demo: `k8s/gateway/demo-envoy.yaml`

---

### Cilium Gateway (`demo.cilium.lab…`, `.203`)

| # | 항목 | 리소스·메모 | 상태 |
| --- | --- | --- | --- |
| 1 | HTTPS | Gateway listener TLS + Secret | 📋 |
| 2 | gRPC | GRPCRoute (Cilium 버전 호환 확인) | 📋 |
| 3 | WebSocket | HTTPRoute | 📋 |
| 4 | Rewrite | HTTPRoute filter 지원 범위 확인 | 📋 |
| 5 | Header | filter / CiliumNetworkPolicy L7 연계 | 📋 |
| 6 | RateLimit | Cilium/BPF rate limit 또는 Envoy layer | 📋 |
| 7 | JWT | Gateway L7 policy 또는 외부 IdP | 📋 |

values: `helm/values/cilium.yaml` (`gatewayAPI.enabled: true`) · demo: `k8s/gateway/demo-cilium.yaml`

---

### Istio Gateway (`demo.istio.lab…`, `.206`)

| # | 항목 | 리소스·메모 | 상태 |
| --- | --- | --- | --- |
| 1 | HTTPS | Gateway API TLS + 또는 `Gateway`/`VirtualService` | 📋 |
| 2 | gRPC | GRPCRoute / VirtualService | 📋 |
| 3 | WebSocket | VirtualService | 📋 |
| 4 | Rewrite | VirtualService `HTTPRewrite` | 📋 |
| 5 | Header | VirtualService headers | 📋 |
| 6 | RateLimit | EnvoyFilter / local rate limit | 📋 |
| 7 | JWT | `RequestAuthentication` + `AuthorizationPolicy` · Keycloak 연동 📋 | 📋 |

mesh mTLS(east-west): [`mtls.md`](mtls.md) — 엣지 JWT와 별개.

---

### Kong (`demo.kong.lab…`, `.204`)

| # | 항목 | Kong 리소스·플러그인 | 상태 |
| --- | --- | --- | --- |
| 1 | HTTPS | Ingress TLS secret / KongCertificate | 📋 |
| 2 | gRPC | Ingress grpc-backend | 📋 |
| 3 | WebSocket | route protocol | 📋 |
| 4 | Rewrite | request-transformer | 📋 |
| 5 | Header | correlation-id, response-transformer | 📋 |
| 6 | RateLimit | rate-limiting plugin | 📋 |
| 7 | JWT | jwt / openid-connect plugin · Keycloak | 📋 |

---

### Traefik (`demo.traefik.lab…`, `.205`)

| # | 항목 | 메모 | 상태 |
| --- | --- | --- | --- |
| 1–7 | 동일 축 | IngressRoute / Middleware (rateLimit, headers, stripPrefix) | 📋 |

---

### ingress-nginx (`.201`) · HAProxy (`.207`)

Ingress v1 annotations / CRD로 동일 7항목 📋. Gateway API 전환 시 Envoy/Cilium/NGINX GF 우선.

---

## 한 줄 검증 (HTTP — Phase 7 기준)

```bash
curl -sS -H 'Host: demo.gateway.lab.origemite.com' http://172.18.255.202/
curl -sS -H 'Host: demo.cilium.lab.origemite.com' http://172.18.255.203/
curl -sS -H 'Host: demo.istio.lab.origemite.com' http://172.18.255.206/
```

HTTPS 추가 후:

```bash
curl -sS -k -H 'Host: demo.gateway.lab.origemite.com' https://172.18.255.202/
```

## 관련

- 7종 일괄 설치: [`ingress-compare.md`](ingress-compare.md)
- 비교 표: [`doc/compare/gateway-compare.md`](../../doc/compare/gateway-compare.md)
- Keycloak(OIDC): [`keycloak.md`](keycloak.md)
