# helm ops

실제 릴리스·차트·네임스페이스 이름이 들어간 복붙 명령만 둔다.

ingress-nginx를 `nginx` 네임스페이스에 설치한다. 저장소 루트에서 실행한다.
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update ingress-nginx
helm upgrade --install nginx ingress-nginx/ingress-nginx \
  -n nginx \
  --create-namespace \
  -f helm/values/nginx.yaml \
  --wait \
  --timeout 10m
```

Argo CD를 `argocd` 네임스페이스에 설치하고 `argocd.lab.origemite.com`으로 연다. nginx 설치 후에 실행한다.
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update argo
helm upgrade --install argocd argo/argo-cd \
  -n argocd \
  --create-namespace \
  --set server.ingress.enabled=true \
  --set server.ingress.ingressClassName=nginx \
  --set server.ingress.hostname=argocd.lab.origemite.com \
  --wait \
  --timeout 10m
```
