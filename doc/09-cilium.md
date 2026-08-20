# 09 Cilium

## 목표

eBPF 기반 Cilium CNI로 **파드 간 ingress 권한**을 `CiliumNetworkPolicy`로 제어한다. Hubble로 허용, 차단 흐름을 관측한다.

## 사전조건

- k3d `lab` 클러스터가 Cilium용으로 재생성됨 (`--disable-network-policy`). [`../cli/ops/cilium.md`](../cli/ops/cilium.md)
- Helm으로 Cilium 설치 완료. [`../helm/values/cilium.yaml`](../helm/values/cilium.yaml) — Hubble relay, UI `enabled: true`
- `cilium status` 또는 `kubectl -n kube-system get pods -l app.kubernetes.io/name=cilium-agent` Ready

## 절차

1. **Cilium 설치** — [`../cli/ops/cilium.md`](../cli/ops/cilium.md)의 클러스터 재생성, Helm 블록 실행.
2. **Hubble UI** — `kubectl -n kube-system port-forward svc/hubble-ui 12000:80` 후 `http://127.0.0.1:12000`.
3. **데모 워크로드** — `cilium-policy` NS에 echo(agnhost HTTP 8080)와 client(busybox) 배포.
 - [`../k8s/namespace/cilium-policy.yaml`](../k8s/namespace/cilium-policy.yaml)
 - [`../k8s/deployment/cilium-policy-echo.yaml`](../k8s/deployment/cilium-policy-echo.yaml)
 - [`../k8s/deployment/cilium-policy-client.yaml`](../k8s/deployment/cilium-policy-client.yaml)
 - [`../k8s/service/cilium-policy-echo.yaml`](../k8s/service/cilium-policy-echo.yaml)
4. **정책 없음** — client에서 `wget -qO- echo:80` 성공(기본 allow).
5. **default-deny ingress** — [`../k8s/ciliumnetworkpolicy/default-deny-ingress.yaml`](../k8s/ciliumnetworkpolicy/default-deny-ingress.yaml) 적용. `ingress: []`로 해당 NS 전체 ingress 차단. client → echo 실패.
6. **선택적 allow** — [`../k8s/ciliumnetworkpolicy/allow-client-to-echo.yaml`](../k8s/ciliumnetworkpolicy/allow-client-to-echo.yaml) 적용. `app: client` → `app: echo` TCP 8080만 허용. client → echo 성공.
7. **Hubble 관측** — `hubble observe --namespace cilium-policy`, 차단 시 `--verdict Dropped` 필터.

## 캡처

<!-- ![Cilium agent Ready](images/09-cilium/cilium-agent-ready.png) -->
<!-- ![Hubble UI flow](images/09-cilium/hubble-ui-flow.png) -->
<!-- ![wget before deny](images/09-cilium/wget-before-deny.png) -->
<!-- ![wget after deny timeout](images/09-cilium/wget-after-deny.png) -->
<!-- ![hubble observe dropped](images/09-cilium/hubble-dropped.png) -->
<!-- ![allow policy wget ok](images/09-cilium/wget-after-allow.png) -->

## 에러, 트러블슈팅

| 증상 | 원인 | 조치 |
| --- | --- | --- |
| Cilium Pod Pending / CrashLoop | k3s 기본 flannel, network-policy와 충돌 | 클러스터 `--disable-network-policy`로 재생성, `K3D_FIX_MOUNTS=1` |
| 정책 적용 후에도 전부 허용 | k3s NetworkPolicy가 여전히 활성 | k3d recreate 후 Cilium만 CNI로 동작하는지 확인 |
| `wget` 타임아웃(의도된 차단) | `default-deny-ingress` 정상 동작 | `allow-client-to-echo` 적용 또는 Hubble `Dropped` 확인 |
| Hubble UI 빈 화면 | relay 미기동, 포트포워드 끊김 | `kubectl -n kube-system get pods -l k8s-app=hubble-relay`; port-forward 재실행 |
| echo Not Ready | agnhost 기동 지연 | `kubectl -n cilium-policy describe pod -l app=echo`; readiness probe 대기 |

## 관련

- [`../cli/ops/cilium.md`](../cli/ops/cilium.md)
- [`../helm/values/cilium.yaml`](../helm/values/cilium.yaml)
- [`../k8s/ciliumnetworkpolicy/`](../k8s/ciliumnetworkpolicy/) — CiliumNetworkPolicy 매니페스트
- [`../k8s/namespace/cilium-policy.yaml`](../k8s/namespace/cilium-policy.yaml)
- [`../k8s/deployment/cilium-policy-echo.yaml`](../k8s/deployment/cilium-policy-echo.yaml)
- [`../k8s/deployment/cilium-policy-client.yaml`](../k8s/deployment/cilium-policy-client.yaml)
- [`../k8s/service/cilium-policy-echo.yaml`](../k8s/service/cilium-policy-echo.yaml)
- [`../k8s/gateway/demo-cilium.yaml`](../k8s/gateway/demo-cilium.yaml) — Gateway API (별도 Phase 7)
