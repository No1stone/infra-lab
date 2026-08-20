# 04 카나리 배포

## 목표

신규 버전에 **일부 트래픽만** 보내 오류·latency를 관측한 뒤 비율을 늘리거나 롤백한다. nginx canary annotation, Istio VirtualService, (계획) Argo Rollouts 중 하나를 랩에서 비교한다.

## 사전조건

- blue/green과 동일 — [`doc/scenarios/03-blue-green.md`](03-blue-green.md)
- 관측 스택 — [`cli/ops/otel-trace-lab.md`](../../cli/ops/otel-trace-lab.md), Prometheus
- (Istio 경로) `ingress-compare` + Istio Gateway — [`cli/ops/istio.md`](../../cli/ops/istio.md)

## 절차

### A — nginx Ingress canary (가벼움)

1. `demo-echo`(stable) + `demo-echo-canary`(1 replica) 배포
2. Ingress `demo-nginx`에 canary annotation — weight 10%
   ```bash
   kubectl -n ingress-compare annotate ingress demo-nginx \
     nginx.ingress.kubernetes.io/canary=true \
     nginx.ingress.kubernetes.io/canary-weight=10 \
     --overwrite
   ```
3. canary 전용 Ingress 또는 secondary service backend 연결 (chart/nginx 버전에 맞게 매니페스트 조정)
4. 다수 curl — 응답 본문 비율로 90/10 근사 확인

### B — Istio VirtualService (mesh)

1. sidecar injection — [`cli/ops/mtls.md`](../../cli/ops/mtls.md)
2. VirtualService `demo-echo`: subset `v1` 90%, `v2` 10%
3. Kiali graph에서 v2 traffic 비율 확인

### C — Argo Rollouts

1. [`cli/ops/argo-rollouts.md`](../../cli/ops/argo-rollouts.md) — 컨트롤러 설치 + `ops-rollouts` Canary (10→30→50%)
2. Prometheus AnalysisTemplate(선택) — 실패 시 자동 rollback
3. `kubectl argo rollouts get rollout …` 로 step 진행 확인

## 검증

- curl 통계: stable vs canary 응답 count
- Prometheus: `sum by (version)(rate(...))` — canary label 필요
- Tempo: canary pod `service.version` span 비율
- canary 실패 시 weight 0 또는 rollback — [`cli/doc/kube.md`](../../cli/doc/kube.md) rollout undo

## 관련

- [`doc/scenarios/03-blue-green.md`](03-blue-green.md)
- [`cli/ops/argo-rollouts.md`](../../cli/ops/argo-rollouts.md)
- [`cli/ops/ingress-compare.md`](../../cli/ops/ingress-compare.md)
- [`cli/ops/istio.md`](../../cli/ops/istio.md)
- HPA와 병행: `k8s/horizontalpodautoscaler/ops-hpa-web.yaml`, `k8s/namespace/ops-hpa.yaml`
