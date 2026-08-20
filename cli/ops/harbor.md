# harbor ops

Harbor 레지스트리. 호스트 `harbor.nginx.lab.origemite.com`. 관리자 비밀번호는 로컬에서만 입력한다.

네임스페이스 적용
```bash
kubectl apply -f k8s/namespace/harbor.yaml
```

관리자 Secret — 비밀번호는 로컬에서만 입력 (`$HARBOR_ADMIN_PASSWORD`에 설정)
```bash
kubectl create secret generic harbor-admin \
  -n harbor \
  --from-literal=HARBOR_ADMIN_PASSWORD="$HARBOR_ADMIN_PASSWORD"
```

Harbor Helm repo 등록 (최초 1회)
```bash
helm repo add harbor https://helm.goharbor.io
helm repo update
```

Harbor 설치 (harbor) — harbor.nginx.lab.origemite.com
```bash
helm upgrade --install harbor harbor/harbor \
  -n harbor \
  -f helm/values/harbor.yaml \
  --wait \
  --timeout 15m
```
