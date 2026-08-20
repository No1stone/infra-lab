# 10 게이트웨이 마이그레이션

## 목표

기본 입구 **nginx**에서 다른 게이트웨이(Istio·Kong·Envoy Gateway 등)로 **호스트·MetalLB·Ingress/Gateway 리소스**를 옮기고, DNS(프록시)와 3뎁스 규칙을 유지한다.

## 사전조건

- 7종 ingress-compare 설치 — [`cli/ops/ingress-compare.md`](../../cli/ops/ingress-compare.md)
- MetalLB `.201`–`.207` — [`cli/ops/metallb.md`](../../cli/ops/metallb.md)
- 호스트 규칙: `<앱>.<게이트웨이>.lab.origemite.com` — [`dns/inventory.yaml`](../../dns/inventory.yaml)

## 절차

1. **현재 매핑 표** — inventory `platform_hosts` vs `phase7_demo_hosts`
2. **데모 앱 동일 backend** — `demo-echo` Service는 공유, 진입만 변경
3. **시나리오: nginx → Istio**
   - 기존: `demo.nginx.lab` @ `.201` — `k8s/ingress/demo-nginx.yaml`
   - 대상: `demo.istio.lab` @ `.206` — `k8s/gateway/demo-istio.yaml`
   ```bash
   curl -sS -H 'Host: demo.nginx.lab.origemite.com' http://172.18.255.201/
   curl -sS -H 'Host: demo.istio.lab.origemite.com' http://172.18.255.206/
   ```
4. **프록시 Host 헤더 전환** — [`cli/ops/proxy.md`](../../cli/ops/proxy.md) — 외부 DNS는 수동, 랩 내부는 MetalLB+Host
5. **TLS** — 새 Gateway에 Certificate/Ingress TLS secret — [`doc/scenarios/08-certificate-renewal.md`](08-certificate-renewal.md)
6. **mTLS mesh** — migration 후 sidecar injection — [`cli/ops/mtls.md`](../../cli/ops/mtls.md)
7. **관측** — gateway별 access log / trace service.name 분리 — [`cli/ops/otel-trace-lab.md`](../../cli/ops/otel-trace-lab.md)
8. **롤백** — 프록시 Host를 nginx로 되돌림

## 검증

- 7개 demo Host 각 200 + body `ingress-compare-demo`
- Kiali: inbound gateway별 traffic
- (선택) 동일 FQDN을 두 controller에 **동시에** 두지 않음 — 3뎁스·게이트웨이 segment로 분리
- Prometheus: per-ingress request metrics

## 관련

- [`doc/14-ingress-compare.md`](../14-ingress-compare.md)
- [`cli/ops/ingress-compare.md`](../../cli/ops/ingress-compare.md)
- [`cli/ops/dns-subzone.md`](../../cli/ops/dns-subzone.md)
- [`doc/scenarios/03-blue-green.md`](03-blue-green.md)
- [`doc/scenarios/07-network-failure.md`](07-network-failure.md)
