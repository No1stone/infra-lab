# istio ops

Istio base → istiod → Kiali 순서. 네임스페이스는 k8s/namespace 매니페스트로 미리 만든다.

Istio, Kiali Helm repo 등록 (최초 1회)
```bash
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo add kiali https://kiali.io/helm-charts
helm repo update
```

네임스페이스 적용
```bash
kubectl apply -f k8s/namespace/istio-system.yaml
kubectl apply -f k8s/namespace/kiali.yaml
```

istio-base 설치 (istio-system)
```bash
helm upgrade --install istio-base istio/base \
 -n istio-system \
 --wait \
 --timeout 10m
```

istiod 설치 (istio-system)
```bash
helm upgrade --install istiod istio/istiod \
 -n istio-system \
 -f helm/values/istiod.yaml \
 --wait \
 --timeout 10m
```

Kiali 설치 (kiali)
```bash
helm upgrade --install kiali-server kiali/kiali-server \
 -n kiali \
 -f helm/values/kiali.yaml \
 --wait \
 --timeout 10m
```

데모 네임스페이스에 sidecar 자동 주입 라벨 (bookinfo 예시)
```bash
kubectl create namespace bookinfo --dry-run=client -o yaml | kubectl apply -f -
kubectl label namespace bookinfo istio-injection=enabled --overwrite
```
