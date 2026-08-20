# helm ops

저장소 루트에서 실행. Helm 리포 등록(최초 1회)
```bash
helm repo add metallb https://metallb.github.io/metallb
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo add jetstack https://charts.jetstack.io
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo add fluent https://fluent.github.io/helm-charts
helm repo add argo https://argoproj.github.io/argo-helm
helm repo add headlamp https://kubernetes-sigs.github.io/headlamp/
helm repo add opensearch https://opensearch-project.github.io/helm-charts/
helm repo update
```

## Phase 1 — MetalLB, nginx, cert-manager

MetalLB 설치 (metallb-system)
```bash
helm upgrade --install metallb metallb/metallb \
  -n metallb-system \
  --create-namespace \
  -f helm/values/metallb.yaml \
  --wait \
  --timeout 10m
```

ingress-nginx 설치 (nginx)
```bash
helm upgrade --install nginx ingress-nginx/ingress-nginx \
  -n nginx \
  --create-namespace \
  -f helm/values/nginx.yaml \
  --wait \
  --timeout 10m
```

cert-manager 설치 (cert-manager)
```bash
helm upgrade --install cert-manager jetstack/cert-manager \
  -n cert-manager \
  --create-namespace \
  -f helm/values/cert-manager.yaml \
  --wait \
  --timeout 10m
```

MetalLB IP 풀·ClusterIssuer 적용 — metallb/cert-manager ops 참고
```bash
kubectl apply -f k8s/configmap/metallb-ip-pool.yaml
kubectl apply -f k8s/configmap/cert-manager-clusterissuer.yaml
```

## Phase 2 — 관측·로그

kube-prometheus-stack 설치 (monitoring)
```bash
helm upgrade --install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  -n monitoring \
  --create-namespace \
  -f helm/values/kube-prometheus-stack.yaml \
  --wait \
  --timeout 10m
```

Loki 설치 (loki)
```bash
helm upgrade --install loki grafana/loki \
  -n loki \
  --create-namespace \
  -f helm/values/loki.yaml \
  --wait \
  --timeout 10m
```

Tempo 설치 (tempo)
```bash
helm upgrade --install tempo grafana/tempo \
  -n tempo \
  --create-namespace \
  -f helm/values/tempo.yaml \
  --wait \
  --timeout 10m
```

OpenTelemetry Collector 설치 (otel)
```bash
helm upgrade --install otel-collector open-telemetry/opentelemetry-collector \
  -n otel \
  --create-namespace \
  -f helm/values/opentelemetry-collector.yaml \
  --wait \
  --timeout 10m
```

fluent-bit 설치 (logging)
```bash
helm upgrade --install fluent-bit fluent/fluent-bit \
  -n logging \
  --create-namespace \
  -f helm/values/fluent-bit.yaml \
  --wait \
  --timeout 10m
```

## Phase 3 — GitOps·UI

Argo CD 설치 (argocd) — argocd.lab.origemite.com
```bash
helm upgrade --install argocd argo/argo-cd \
  -n argocd \
  --create-namespace \
  -f helm/values/argocd.yaml \
  --wait \
  --timeout 10m
```

Headlamp 설치 (headlamp) — headlamp.lab.origemite.com
```bash
helm upgrade --install headlamp headlamp/headlamp \
  -n headlamp \
  --create-namespace \
  -f helm/values/headlamp.yaml \
  --wait \
  --timeout 10m
```

## Phase 4 (선택) — OpenSearch

OpenSearch 설치 (opensearch)
```bash
helm upgrade --install opensearch opensearch/opensearch \
  -n opensearch \
  --create-namespace \
  -f helm/values/opensearch.yaml \
  --wait \
  --timeout 10m
```

## Phase 6 — Cilium, Istio, Harbor, Keycloak

Phase 6 repo 추가 (최초 1회)
```bash
helm repo add cilium https://helm.cilium.io/
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo add kiali https://kiali.io/helm-charts
helm repo add harbor https://helm.goharbor.io
helm repo add codecentric https://codecentric.github.io/helm-charts
helm repo update
```

Cilium 설치 — lab 재생성 후 kube-system (cilium ops 참고)
```bash
helm upgrade --install cilium cilium/cilium \
  -n kube-system \
  -f helm/values/cilium.yaml \
  --wait \
  --timeout 10m
```

istio-base 설치 (istio-system)
```bash
helm upgrade --install istio-base istio/base \
  -n istio-system \
  --create-namespace \
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
  --create-namespace \
  -f helm/values/kiali.yaml \
  --wait \
  --timeout 10m
```

Harbor 설치 (harbor) — harbor.lab.origemite.com, harbor ops에서 Secret 선행
```bash
helm upgrade --install harbor harbor/harbor \
  -n harbor \
  --create-namespace \
  -f helm/values/harbor.yaml \
  --wait \
  --timeout 15m
```

Keycloakx 설치 (keycloak) — keycloak.lab.origemite.com, keycloak ops에서 Secret 선행
```bash
helm upgrade --install keycloak codecentric/keycloakx \
  -n keycloak \
  --create-namespace \
  -f helm/values/keycloak.yaml \
  --wait \
  --timeout 15m
```

## Phase 7 — 진입점 비교 (ingress-compare)

Phase 7 repo 추가 (최초 1회)
```bash
helm repo add kong https://charts.konghq.com
helm repo add traefik https://traefik.github.io/charts
helm repo add haproxytech https://haproxytech.github.io/helm-charts
helm repo update
```

ingress-nginx IP 고정 (nginx) — demo.nginx.lab.origemite.com, MetalLB .201
```bash
helm upgrade --install nginx ingress-nginx/ingress-nginx \
  -n nginx \
  --create-namespace \
  -f helm/values/nginx.yaml \
  --wait \
  --timeout 10m
```

Envoy Gateway OCI — demo.gateway.lab.origemite.com, MetalLB .202
```bash
helm upgrade --install eg oci://docker.io/envoyproxy/gateway-helm \
  -n envoy-gateway-system \
  --create-namespace \
  -f helm/values/envoy-gateway.yaml \
  --wait \
  --timeout 10m
```

Cilium Gateway API — demo.cilium.lab.origemite.com, MetalLB .203
```bash
helm upgrade cilium cilium/cilium \
  -n kube-system \
  -f helm/values/cilium.yaml \
  --wait \
  --timeout 10m
```

Kong — demo.kong.lab.origemite.com, MetalLB .204
```bash
helm upgrade --install kong kong/kong \
  -n kong \
  --create-namespace \
  -f helm/values/kong.yaml \
  --wait \
  --timeout 10m
```

Traefik — demo.traefik.lab.origemite.com, MetalLB .205
```bash
helm upgrade --install traefik traefik/traefik \
  -n traefik \
  --create-namespace \
  -f helm/values/traefik.yaml \
  --wait \
  --timeout 10m
```

Istio Gateway — demo.istio.lab.origemite.com, MetalLB .206
```bash
helm upgrade --install istio-gateway istio/gateway \
  -n istio-system \
  -f helm/values/istio-gateway.yaml \
  --wait \
  --timeout 10m
```

HAProxy Ingress — demo.haproxy.lab.origemite.com, MetalLB .207
```bash
helm upgrade --install haproxy-ingress haproxytech/kubernetes-ingress \
  -n haproxy-ingress \
  --create-namespace \
  -f helm/values/haproxy-ingress.yaml \
  --wait \
  --timeout 10m
```

데모 매니페스트·curl — [`cli/ops/ingress-compare.md`](ingress-compare.md)

lab 클러스터 Helm 릴리스 전체 확인
```bash
helm list -A --kube-context k3d-lab
```
