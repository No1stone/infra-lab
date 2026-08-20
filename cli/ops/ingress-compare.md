# ingress-compare ops

Phase 7 — **7개 진입점 컨트롤러를 한 번에** 올리고 `demo-echo`로 비교한다.

매니페스트·values는 이미 모두 있다. 아래를 순서대로 실행하면 된다.

| # | 컨트롤러 | values | demo | MetalLB |
| --- | --- | --- | --- | --- |
| 1 | ingress-nginx | `helm/values/nginx.yaml` | `k8s/ingress/demo-nginx.yaml` | .201 |
| 2 | Envoy Gateway | `helm/values/envoy-gateway.yaml` | `k8s/gateway/demo-envoy.yaml` | .202 |
| 3 | Cilium Gateway | `helm/values/cilium.yaml` | `k8s/gateway/demo-cilium.yaml` | .203 |
| 4 | Kong | `helm/values/kong.yaml` | `k8s/ingress/demo-kong.yaml` | .204 |
| 5 | Traefik | `helm/values/traefik.yaml` | `k8s/ingress/demo-traefik.yaml` | .205 |
| 6 | Istio Gateway | `helm/values/istio-gateway.yaml` | `k8s/gateway/demo-istio.yaml` | .206 |
| 7 | HAProxy Ingress | `helm/values/haproxy-ingress.yaml` | `k8s/ingress/demo-haproxy.yaml` | .207 |

호스트: `demo.<게이트웨이>.lab.origemite.com` (3뎁스). 동시 기동 idle 약 **+4GiB**.

## 사전조건

- MetalLB + IP pool (`.201`–`.207` 포함)
- Cilium 설치됨 (`gatewayAPI.enabled: true` — values에 있음)
- Istio `istiod` 설치됨 (Istio Gateway용)
- 프록시 DNS는 외부. 랩 안은 Host 헤더 + MetalLB로 충분

## 1) Gateway API CRD (최초 1회)

```bash
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.1/standard-install.yaml
```

## 2) Helm repo

```bash
helm repo add kong https://charts.konghq.com
helm repo add traefik https://traefik.github.io/charts
helm repo add haproxytech https://haproxytech.github.io/helm-charts
helm repo update
```

## 3) 컨트롤러 7개 한 번에 설치

저장소 루트에서. 이미 있으면 upgrade.

```bash
helm upgrade --install nginx ingress-nginx/ingress-nginx \
  -n nginx --create-namespace \
  -f helm/values/nginx.yaml --wait --timeout 10m

helm upgrade --install eg oci://docker.io/envoyproxy/gateway-helm \
  -n envoy-gateway-system --create-namespace \
  -f helm/values/envoy-gateway.yaml --wait --timeout 10m

helm upgrade cilium cilium/cilium \
  -n kube-system \
  -f helm/values/cilium.yaml --wait --timeout 10m

helm upgrade --install kong kong/kong \
  -n kong --create-namespace \
  -f helm/values/kong.yaml --wait --timeout 10m

helm upgrade --install traefik traefik/traefik \
  -n traefik --create-namespace \
  -f helm/values/traefik.yaml --wait --timeout 10m

helm upgrade --install istio-gateway istio/gateway \
  -n istio-system \
  -f helm/values/istio-gateway.yaml --wait --timeout 10m

helm upgrade --install haproxy-ingress haproxytech/kubernetes-ingress \
  -n haproxy-ingress --create-namespace \
  -f helm/values/haproxy-ingress.yaml --wait --timeout 10m
```

## 4) 데모 앱 + Ingress/Gateway 7종

```bash
kubectl apply -f k8s/namespace/ingress-compare.yaml
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

Envoy/Cilium Gateway가 만든 LB Service IP가 비면 MetalLB로 고정.

```bash
kubectl -n ingress-compare annotate svc --overwrite \
  -l gateway.envoyproxy.io/owning-gateway-name=demo-envoy \
  metallb.universe.tf/loadBalancerIPs=172.18.255.202 2>/dev/null || true

kubectl get svc -A -o wide | grep -E 'LoadBalancer|172\.18\.255\.20'
```

## 5) 한 번에 curl 테스트

```bash
curl -sS -H 'Host: demo.nginx.lab.origemite.com' http://172.18.255.201/
curl -sS -H 'Host: demo.gateway.lab.origemite.com' http://172.18.255.202/
curl -sS -H 'Host: demo.cilium.lab.origemite.com' http://172.18.255.203/
curl -sS -H 'Host: demo.kong.lab.origemite.com' http://172.18.255.204/
curl -sS -H 'Host: demo.traefik.lab.origemite.com' http://172.18.255.205/
curl -sS -H 'Host: demo.istio.lab.origemite.com' http://172.18.255.206/
curl -sS -H 'Host: demo.haproxy.lab.origemite.com' http://172.18.255.207/
```

기대 본문: `ingress-compare-demo`

## 관련

- values·개별 helm: [`helm.md`](helm.md) Phase 7
- 프록시 Host: [`proxy.md`](proxy.md)
- DNS 이름: [`dns/inventory.yaml`](../../dns/inventory.yaml)
