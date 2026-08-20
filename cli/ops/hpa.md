# hpa ops

HorizontalPodAutoscaler(HPA)로 CPU 부하에 따라 `ops-hpa-web` replica를 자동 확장하는 실습. `metrics-server` 필요.

## metrics-server (k3d)

HPA, `kubectl top`은 metrics-server가 없으면 동작하지 않는다. k3s/k3d는 기본 포함되지 않을 수 있음.

공식 manifest + k3d kubelet TLS 우회:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl -n kube-system rollout status deployment/metrics-server
kubectl -n kube-system patch deployment metrics-server --type='json' \
 -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
kubectl -n kube-system rollout status deployment/metrics-server
kubectl top nodes
kubectl top pods -A
```

Helm 대안 (kubernetes-sigs chart):

```bash
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm repo update
helm upgrade --install metrics-server metrics-server/metrics-server \
 -n kube-system \
 --set args="{--kubelet-insecure-tls}" \
 --wait \
 --timeout 5m
```

## 워크로드, HPA 적용

```bash
kubectl apply -f k8s/namespace/ops-hpa.yaml
kubectl apply -f k8s/deployment/ops-hpa-web.yaml
kubectl apply -f k8s/service/ops-hpa-web.yaml
kubectl apply -f k8s/horizontalpodautoscaler/ops-hpa-web.yaml
kubectl -n ops-hpa rollout status deployment/ops-hpa-web
kubectl -n ops-hpa get hpa
kubectl -n ops-hpa describe hpa ops-hpa-web
```

HPA 상태 관측 (별도 터미널):

```bash
watch kubectl -n ops-hpa get hpa,pods
```

## 부하 생성

포트 포워드:

```bash
kubectl -n ops-hpa port-forward svc/ops-hpa-web 8081:80
```

### hey (권장)

설치 (Ubuntu 노트북, Go 없을 때):

```bash
go install github.com/rakyll/hey@latest
# 또는
sudo apt-get update && sudo apt-get install -y hey
```

부하 (CPU request 100m, target 50% → 약 50m 사용 시 scale-out 기대):

```bash
hey -z 3m -c 50 http://127.0.0.1:8081/
```

### k6 대안

```bash
kubectl run k6-load -n ops-hpa --rm -i --restart=Never \
 --image=grafana/k6:0.54.0 -- \
 run - --vus 30 --duration 3m <<'EOF'
import http from 'k6/http';
export default function () {
 http.get('http://ops-hpa-web.ops-hpa.svc.cluster.local/');
}
EOF
```

클러스터 내부에서 Service DNS로 직접 때리므로 port-forward 불필요.

## 스케일 확인

```bash
kubectl -n ops-hpa get hpa ops-hpa-web
kubectl -n ops-hpa get pods -o wide
kubectl top pods -n ops-hpa
kubectl -n ops-hpa describe hpa ops-hpa-web | tail -20
```

부하 중단 후 scale-down (기본 stabilization ~5분):

```bash
kubectl -n ops-hpa get hpa -w
```

## Grafana CPU 확인

kube-prometheus-stack 설치 후 Grafana에서 CPU 사용률 대조.

- URL: https://grafana.nginx.lab.origemite.com
- Explore → Prometheus: `rate(container_cpu_usage_seconds_total{namespace="ops-hpa", pod=~"ops-hpa-web.*"}[1m])`
- 또는 Dashboards → Kubernetes / Compute Resources → Namespace: `ops-hpa`

Helm 미설치 시 — [`cli/ops/helm.md`](helm.md) Phase 2 `kube-prometheus-stack` 참고.

## kubectl autoscale (YAML 대신)

동일 설정을 imperative로:

```bash
kubectl autoscale deployment ops-hpa-web -n ops-hpa \
 --cpu-percent=50 --min=1 --max=5
```

YAML manifest와 중복 적용하지 말 것. 비교 후 정리:

```bash
kubectl delete hpa ops-hpa-web -n ops-hpa
kubectl apply -f k8s/horizontalpodautoscaler/ops-hpa-web.yaml
```

## 정리

```bash
kubectl delete namespace ops-hpa
```
