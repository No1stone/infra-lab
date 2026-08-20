# 08 인증서 갱신

## 목표

cert-manager **Certificate** lifecycle(발급·갱신·만료)을 이해하고, Ingress TLS secret rollover 시 **다운타임 없이** HTTPS가 유지되는지 확인한다.

## 사전조건

- cert-manager + selfsigned ClusterIssuer — [`cli/ops/cert-manager.md`](../../cli/ops/cert-manager.md)
- nginx Ingress — [`helm/values/nginx.yaml`](../../helm/values/nginx.yaml)
- TLS Ingress 예: `k8s/ingress/lab.yaml`, platform hosts — [`dns/inventory.yaml`](../../dns/inventory.yaml)

## 절차

1. **Issuer·Certificate 상태**
   ```bash
   kubectl get clusterissuer
   kubectl get certificate -A
   kubectl describe certificate -n nginx
   ```
2. **Secret 확인** — `tls.crt` / `tls.key`, `notAfter`
   ```bash
   kubectl get secret -n nginx -l cert-manager.io/certificate-name
   kubectl get secret <tls-secret> -n nginx -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -noout -dates
   ```
3. **강제 재발급** — annotation 또는 Certificate spec `duration`/`renewBefore` 조정
   ```bash
   kubectl cert-manager renew <certificate-name> -n <namespace>
   ```
   (CLI 미설치 시 Certificate delete 후 Ingress annotation 재트리거)
4. **Ingress reload** — nginx controller가 secret mount 갱신하는지 controller log 확인
5. **(실습) 짧은 duration** — test Certificate `duration: 1h`, `renewBefore: 30m`으로 갱신 이벤트 관측
6. **엣지 TLS vs mesh mTLS** — Ingress cert는 **north-south**, Istio cert는 **east-west** — [`doc/11-mtls.md`](../11-mtls.md)

## 검증

- `curl -vIk https://grafana.nginx.lab.origemite.com` (또는 Host + MetalLB IP) — cert dates 변경
- `kubectl describe certificaterequest -n nginx`
- Prometheus: cert-manager certificate expiry metrics (exporter 활성 시)
- 갱신 전후 curl 연속 성공

## 관련

- [`cli/ops/cert-manager.md`](../../cli/ops/cert-manager.md)
- [`doc/04-metallb-ingress-cert.md`](../04-metallb-ingress-cert.md)
- [`helm/values/cert-manager.yaml`](../../helm/values/cert-manager.yaml)
- [`doc/scenarios/09-secret-rotation.md`](09-secret-rotation.md)
