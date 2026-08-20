# argo-rollouts ops

Argo Rollouts Canary(10→30→50%), Blue-Green, 롤백 실습. 네임스페이스 `ops-rollouts`.

## 1) Rollouts controller 설치

### kubectl (공식 manifest)

```bash
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
kubectl -n argo-rollouts rollout status deployment/argo-rollouts
```

### Helm (대안)

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm upgrade --install argo-rollouts argo/argo-rollouts \
 -n argo-rollouts \
 --create-namespace \
 --wait \
 --timeout 5m
```

CLI 플러그인 (선택, promote/status 편의).

```bash
curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-darwin-arm64
chmod +x kubectl-argo-rollouts-darwin-arm64
sudo mv kubectl-argo-rollouts-darwin-arm64 /usr/local/bin/kubectl-argo-rollouts
kubectl argo rollouts version
```

## 2) 데모 Rollout 배포

```bash
kubectl apply -f k8s/namespace/ops-rollouts.yaml
kubectl apply -f k8s/rollout/rollouts-demo-services.yaml
kubectl apply -f k8s/rollout/rollouts-demo.yaml
kubectl -n ops-rollouts get rollout,rs,pod
kubectl argo rollouts get rollout rollouts-demo -n ops-rollouts
```

## 3) Canary — 10 → 30 → 50 %

새 버전 트리거 (`:blue` → `:green`).

```bash
kubectl argo rollouts set image rollouts-demo \
 rollouts-demo=argoproj/rollouts-demo:green \
 -n ops-rollouts
kubectl argo rollouts get rollout rollouts-demo -n ops-rollouts --watch
```

각 `pause` 단계에서 가중치 확인 후 수동 승격.

```bash
kubectl argo rollouts status rollouts-demo -n ops-rollouts
kubectl argo rollouts promote rollouts-demo -n ops-rollouts
```

10% → pause → 30% → pause → 50% → pause → 100% 순서. `promote`를 pause마다 반복.

트래픽 분배 확인 (stable/canary Service selector는 Rollout이 ReplicaSet 라벨로 관리).

```bash
kubectl -n ops-rollouts get rs -l app=rollouts-demo -o wide
kubectl -n ops-rollouts port-forward svc/rollouts-demo-stable 8080:80 &
curl -s localhost:8080/color
kill %1
```

## 4) Prometheus 메트릭 분석 (선택 / stub)

kube-prometheus-stack이 있으면 AnalysisTemplate으로 Canary 게이트를 둘 수 있다. 아래는 **stub** — 실제 PromQL, 임계값은 환경에 맞게 조정.

```yaml
# k8s/rollout/analysis-success-rate.yaml (참고용, 기본 Rollout에는 미연결)
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
 name: success-rate
 namespace: ops-rollouts
spec:
 metrics:
 - name: success-rate
 interval: 30s
 count: 3
 successCondition: result[0] >= 0.95
 failureLimit: 1
 provider:
 prometheus:
 address: http://kube-prometheus-stack-prometheus.monitoring.svc:9090
 query: |
 sum(rate(http_requests_total{job="rollouts-demo",status=~"2.."}[1m]))
 /
 sum(rate(http_requests_total{job="rollouts-demo"}[1m]))
```

Rollout `strategy.canary.steps`에 `- analysis: { templates: [{ templateName: success-rate }] }`를 넣으면 메트릭 통과 시에만 다음 단계로 진행. 메트릭, ServiceMonitor 미구성 시 이 단계는 생략.

## 5) Blue-Green (별도 전략)

동일 앱에 Blue-Green을 쓰려면 Rollout strategy를 교체한다.

```bash
kubectl -n ops-rollouts patch rollout rollouts-demo --type merge -p '
{
 "spec": {
 "strategy": {
 "blueGreen": {
 "activeService": "rollouts-demo-stable",
 "previewService": "rollouts-demo-canary",
 "autoPromotionEnabled": false
 }
 }
 }
}'
```

새 이미지 배포 후 preview에서 확인, 승격.

```bash
kubectl argo rollouts set image rollouts-demo \
 rollouts-demo=argoproj/rollouts-demo:yellow \
 -n ops-rollouts
kubectl -n ops-rollouts port-forward svc/rollouts-demo-canary 8081:80 &
curl -s localhost:8081/color
kill %1
kubectl argo rollouts promote rollouts-demo -n ops-rollouts
```

Canary로 되돌릴 때는 `k8s/rollout/rollouts-demo.yaml`을 다시 apply.

## 6) 롤백

진행 중 Canary 중단, 이전 ReplicaSet으로 되돌리기.

```bash
kubectl argo rollouts abort rollouts-demo -n ops-rollouts
kubectl argo rollouts undo rollouts-demo -n ops-rollouts
kubectl argo rollouts get rollout rollouts-demo -n ops-rollouts
```

이력에서 특정 revision으로 롤백.

```bash
kubectl argo rollouts history rollouts-demo -n ops-rollouts
kubectl argo rollouts undo rollouts-demo --to-revision=1 -n ops-rollouts
```

## 7) 정리

```bash
kubectl delete -f k8s/rollout/rollouts-demo.yaml
kubectl delete -f k8s/rollout/rollouts-demo-services.yaml
kubectl delete -f k8s/namespace/ops-rollouts.yaml
```

## 관련

- Rollout: [`k8s/rollout/rollouts-demo.yaml`](../../k8s/rollout/rollouts-demo.yaml)
- Service: [`k8s/rollout/rollouts-demo-services.yaml`](../../k8s/rollout/rollouts-demo-services.yaml)
- Prometheus stack: [`helm.md`](helm.md) Phase 2
