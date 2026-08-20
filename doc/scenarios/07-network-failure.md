# 07 네트워크 장애

## 목표

**L3/L4 차단, DNS 실패, mTLS 거부, CiliumNetworkPolicy drop**을 재현하고, Hubble/Kiali/Tempo로 원인 구간을 좁힌다.

## 사전조건

- Cilium CNI — [`cli/ops/cilium.md`](../../cli/ops/cilium.md)
- `cilium-policy` 데모 NS
- (선택) Istio STRICT mTLS — [`cli/ops/mtls.md`](../../cli/ops/mtls.md)

## 절차

1. **baseline** — client → echo 성공
 ```bash
 kubectl -n cilium-policy exec deploy/client -- wget -qO- echo:80
 ```
2. **NetworkPolicy default-deny** — [`cli/ops/chaos.md`](../../cli/ops/chaos.md) §5
 ```bash
 kubectl apply -f k8s/ciliumnetworkpolicy/default-deny-ingress.yaml
 kubectl -n cilium-policy exec deploy/client -- wget -qO- --timeout=3 echo:80 || echo BLOCKED
 ```
3. **Hubble observe** — Dropped verdict
 ```bash
 cilium hubble observe --namespace cilium-policy --verdict Dropped
 ```
4. **allow 정책 복구** — `allow-client-to-echo.yaml`
5. **Gateway 단절** — nginx scale 0 — [`cli/ops/chaos.md`](../../cli/ops/chaos.md) §3
6. **east-west mTLS** — PeerAuthentication STRICT + plaintext client 실패 — [`cli/ops/mtls.md`](../../cli/ops/mtls.md)
7. **(선택) 노드 네트워크** — agent docker stop — [`cli/ops/node-failure.md`](../../cli/ops/node-failure.md)

## 검증

- Hubble UI / CLI: drop reason, source/destination labels
- Kiali: edge red, mTLS lock icon
- Prometheus: nginx 502 rate, `kube_pod_status_ready`
- Tempo: client span error / timeout — [`cli/ops/otel-trace-lab.md`](../../cli/ops/otel-trace-lab.md)
- Loki: `{namespace="cilium-policy"} |= "refused"`

## 관련

- [`cli/ops/cilium.md`](../../cli/ops/cilium.md)
- [`cli/ops/chaos.md`](../../cli/ops/chaos.md)
- [`doc/09-cilium.md`](../09-cilium.md)
- [`doc/11-mtls.md`](../11-mtls.md)
- [`doc/scenarios/10-gateway-migration.md`](10-gateway-migration.md)
