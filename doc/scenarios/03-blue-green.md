# 03 블루-그린 배포

## 목표

**blue(현재)** 와 **green(신규)** 두 버전을 동시에 띄운 뒤, Service selector 또는 Ingress backend 전환으로 **한 번에** 트래픽을 옮긴다. 롤백은 selector/Ingress만 되돌리면 된다.

## 사전조건

- `ingress-compare` NS + `demo-echo` — [`cli/ops/ingress-compare.md`](../../cli/ops/ingress-compare.md)
- nginx Ingress + MetalLB `.201`
- (선택) Istio injection — [`cli/ops/mtls.md`](../../cli/ops/mtls.md)

## 절차

1. **blue 배포** — 기존 `demo-echo` (label `version=blue`로 patch)
 ```bash
 kubectl -n ingress-compare label deployment demo-echo version=blue --overwrite
 kubectl -n ingress-compare patch deployment demo-echo -p '{"spec":{"template":{"metadata":{"labels":{"version":"blue"}}}}}'
 ```
2. **green Deployment 복제** — `demo-echo-green`, image/tag 또는 env만 변경, label `version=green`
 ```bash
 kubectl -n ingress-compare get deployment demo-echo -o yaml | \
 sed 's/name: demo-echo/name: demo-echo-green/;s/version: blue/version: green/' | \
 kubectl apply -f -
 ```
 (실습에서는 `kubectl create deployment demo-echo-green --image=…` + 동일 probe/label로 단순화 가능)
3. **Service** — selector `app=demo-echo` 유지, green만 붙일 때는 **selector를 blue/green 공통 app**으로 (`app=demo-echo`, version은 Service subset 또는 별도 Service)
4. **전환** — nginx Ingress `demo-nginx` backend Service를 green Service로 patch, 또는 단일 Service selector를 `version=green`으로 변경
5. **관측** — curl Host `demo.nginx.lab.origemite.com` 본문, 헤더 변경 확인
6. **롤백** — selector `version=blue` 복원, green scale 0

## 검증

- `curl -sS -H 'Host: demo.nginx.lab.origemite.com' http://172.18.255.201/` — blue → green 응답 차이
- `kubectl -n ingress-compare get deploy,svc,endpoints`
- Grafana: nginx request rate 급변 없음 (순간 전환)
- Kiali traffic graph — [`doc/10-istio-kiali.md`](../10-istio-kiali.md)

## 관련

- [`cli/ops/ingress-compare.md`](../../cli/ops/ingress-compare.md)
- [`k8s/ingress/demo-nginx.yaml`](../../k8s/ingress/demo-nginx.yaml)
- [`doc/scenarios/04-canary.md`](04-canary.md)
- [`doc/scenarios/01-zero-downtime-update.md`](01-zero-downtime-update.md)
