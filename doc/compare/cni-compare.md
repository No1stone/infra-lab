# CNI 비교 — Calico, Cilium, Flannel

infra-lab 랩은 **Cilium**을 k3d `lab` CNI로 사용한다(Phase 6A). Calico·Flannel은 **대안 참고**용이다.

## 요약 매트릭스

| 항목 | Calico | Cilium | Flannel |
| --- | --- | --- | --- |
| **eBPF** | ◐ (데이터 plane 옵션) | ★ 기본·kube-proxy 대체 | ❌ userspace/VXLAN 위주 |
| **성능** | 중~높음 (BGP·direct routing 시) | ★ 높음 (eBPF, DSR 등) | 낮음~중 (오버레이) |
| **NetworkPolicy** | ★ Kubernetes NP + Calico NP | ★ CiliumNetworkPolicy (L3–L7) | ❌ (별도 CNI 조합 필요) |
| **Gateway** | ❌ (별도 Ingress/GW) | ★ Gateway API 내장 | ❌ |
| **Hubble** | ❌ (Calico Enterprise 등 별도) | ★ 오픈소스 Hubble | ❌ |
| **Service mesh** | ◐ (WireGuard·Calico mesh) | ◐ Cilium mesh (mutual auth) | ❌ |
| **ops 난이도** | 중 (BGP·IP pool) | 중 (CNI+Gateway+Hubble) | **낮음** |
| **랩 상태** | 📋 개념 | ✅ Phase 6A 실습 | 📋 k3s 기본 CNI(교체 전) |

## eBPF·데이터 경로

| CNI | 데이터 plane | kube-proxy | 메모 |
| --- | --- | --- | --- |
| **Calico** | iptables 또는 eBPF dataplane 선택 | 보통 kube-proxy 유지 | BGP peering, IPAM 유연 |
| **Cilium** | eBPF为主, `kubeProxyReplacement: true` | 대체 가능 ★ | k3d lab values: [`helm/values/cilium.yaml`](../../helm/values/cilium.yaml) |
| **Flannel** | VXLAN/host-gw | kube-proxy 의존 | 설정 최소, Policy 없음 |

**성능(일반론)**: Flannel < Calico(iptables) ≲ Calico(eBPF) < Cilium(eBPF) — 워크load·MTU·직접 라우팅 여부에 따라 달라짐. 홈랩 k3d에서는 **기능·관측** 차이가 체감상 더 크다.

## NetworkPolicy

| CNI | K8s NetworkPolicy | 확장 정책 | L7 |
| --- | --- | --- | --- |
| Calico | ✅ | Calico NetworkPolicy (Global NP 등) | ◐ (WAF 연동·Enterprise) |
| Cilium | ✅ | **CiliumNetworkPolicy** | ★ HTTP method/path, FQDN egress |
| Flannel | ❌ | — | — |

랩 실습:

- [`k8s/ciliumnetworkpolicy/`](../../k8s/ciliumnetworkpolicy/) — default-deny, allow L3/L4, L7 GET-only(초안), FQDN(초안)
- [`cli/ops/cilium.md`](../../cli/ops/cilium.md) — Hubble `Dropped`/`Forwarded` 대조

## Gateway·Ingress (CNI 관점)

| CNI | North-south | 비고 |
| --- | --- | --- |
| Calico | 별도 컨트롤러 필요 | CNI는 L3/L4에 집중 |
| Cilium | **Cilium Gateway** (Gateway API) | Phase 7 `demo.cilium.lab…`. Ingress Controller는 lab에서 off |
| Flannel | 별도 컨트롤러 | k3s 기본만으로는 입구 없음 |

## Hubble·관측

| CNI | Flow log / UI | Prometheus | 랩 |
| --- | --- | --- | --- |
| Calico | Flow logs (설정 필요) | ◐ | — |
| Cilium | **Hubble** CLI·UI ★ | ✅ | `hubble.enabled`, relay·UI on |
| Flannel | ❌ | ❌ | — |

Hubble은 **L3/L4 verdict**와 L7(활성화 시) HTTP 메타데이터를 Pod 간 흐름으로 본다. NetworkPolicy 디버깅에 Cilium이 유리.

## Service mesh와의 관계

| CNI | 내장 mesh | 외부 mesh와 |
| --- | --- | --- |
| Calico | ◐ encryption/mesh 옵션 | Istio sidecar와 병행 가능 |
| Cilium | **Cilium mesh** (SPIRE/SPIFFE, mutual auth) | Istio와 **동시 full mesh 비권장** — 랩은 Istio 선택 |
| Flannel | ❌ | Istio/Linkerd는 CNI 위에만 |

mesh 비교: [`service-mesh-compare.md`](service-mesh-compare.md)

## k3d / k3s 메모

- k3s **기본 CNI**: Flannel. Cilium 전환 시 `--disable-network-policy` + Cilium Helm 필요 — [`cli/ops/cilium.md`](../../cli/ops/cilium.md)
- `K3D_FIX_MOUNTS=1`: k3d volume mount 이슈 회피
- Calico on k3s: 가능하나 이 랩 커리큘럼 **범위 밖** 📋

## 선택 가이드 (초안)

| 목적 | 추천 |
| --- | --- |
| 최소 구성·학습만 | Flannel (k3s 기본) |
| Policy + BGP/on-prem | Calico |
| eBPF + L7 Policy + Hubble + Gateway API | **Cilium** (infra-lab 선택) |

## 관련

- Cilium 튜토리얼: [`doc/09-cilium.md`](../09-cilium.md)
- Cilium ops: [`cli/ops/cilium.md`](../../cli/ops/cilium.md)
- Gateway 비교: [`gateway-compare.md`](gateway-compare.md)
