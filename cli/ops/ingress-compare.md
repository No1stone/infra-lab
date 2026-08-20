# ingress-compare ops

Phase 7 — 7개 진입점 컨트롤러를 동시에 띄워 `demo-echo`로 라우팅을 비교한다.

## DNS·프록시 (서브존)

호스트는 `demo.<컨트롤러>.lab.origemite.com`. Route53은 외부 관리 — 이름만 [`dns/inventory.yaml`](../../dns/inventory.yaml)에 표기.

- DNS 참조·dig: [`dns-subzone.md`](dns-subzone.md)
- 프록시 Host → MetalLB `.201`–`.207`: [`proxy.md`](proxy.md)
- 랩 안만 검증할 때는 MetalLB IP + Host 헤더로 충분하다.

## Gateway API CRD (최초 1회)

```bash
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.1/standard-install.yaml
```

## Helm 설치

리포 추가·업데이트는 [`cli/ops/helm.md`](helm.md) Phase 7 참고.

컨트롤러별 LoadBalancer IP는 MetalLB `.201`–`.207` (`helm/values`에 고정).

## Cilium Gateway API

Cilium values에 `gatewayAPI.enabled: true` 후 upgrade — [`cli/ops/cilium.md`](cilium.md) 참고.

```bash
helm upgrade cilium cilium/cilium \
  -n kube-system \
  -f helm/values/cilium.yaml \
  --wait \
  --timeout 10m
```

## 데모 매니페스트 적용

```bash
kubectl apply -f k8s/namespace/ingress-compare.yaml
kubectl apply -f k8s/namespace/kong.yaml
kubectl apply -f k8s/namespace/traefik.yaml
kubectl apply -f k8s/namespace/haproxy-ingress.yaml
kubectl apply -f k8s/namespace/envoy-gateway-system.yaml
kubectl apply -f k8s/deployment/demo-echo.yaml
kubectl apply -f k8s/service/demo-echo.yaml
kubectl apply -f k8s/ingress/demo-nginx.yaml
kubectl apply -f k8s/ingress/demo-kong.yaml
kubectl apply -f k8s/ingress/demo-traefik.yaml
kubectl apply -f k8s/ingress/demo-haproxy.yaml
kubectl apply -f k8s/gateway/demo-envoy.yaml
kubectl apply -f k8s/gateway/demo-cilium.yaml
kubectl apply -f k8s/gateway/demo-istio.yaml
```

## curl (Host 헤더 → MetalLB IP)

```bash
curl -sS -H 'Host: demo.nginx.lab.origemite.com' http://172.18.255.201/
curl -sS -H 'Host: demo.gateway.lab.origemite.com' http://172.18.255.202/
curl -sS -H 'Host: demo.cilium.lab.origemite.com' http://172.18.255.203/
curl -sS -H 'Host: demo.kong.lab.origemite.com' http://172.18.255.204/
curl -sS -H 'Host: demo.traefik.lab.origemite.com' http://172.18.255.205/
curl -sS -H 'Host: demo.istio.lab.origemite.com' http://172.18.255.206/
curl -sS -H 'Host: demo.haproxy.lab.origemite.com' http://172.18.255.207/
```

기대 응답 본문: `ingress-compare-demo`

## MetalLB IP 표

| IP | 컨트롤러 | 호스트 |
| --- | --- | --- |
| 172.18.255.201 | ingress-nginx | demo.nginx.lab.origemite.com |
| 172.18.255.202 | Envoy Gateway | demo.gateway.lab.origemite.com |
| 172.18.255.203 | Cilium Gateway | demo.cilium.lab.origemite.com |
| 172.18.255.204 | Kong | demo.kong.lab.origemite.com |
| 172.18.255.205 | Traefik | demo.traefik.lab.origemite.com |
| 172.18.255.206 | Istio Gateway | demo.istio.lab.origemite.com |
| 172.18.255.207 | HAProxy Ingress | demo.haproxy.lab.origemite.com |

동시 기동 시 idle 기준 RAM **약 +4GiB** 추가. 32+64 랩에서 Phase 1–6과 병행 가능. Envoy/Cilium Gateway LB IP는 Gateway 생성 후 `kubectl get svc -A | grep LoadBalancer`로 확인하고, 필요하면 Service에 MetalLB annotation으로 `.202`/`.203`을 고정한다.
