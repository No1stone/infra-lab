# keycloak ops

Keycloak IAM. 호스트 `keycloak.nginx.lab.origemite.com`. 관리자 비밀번호는 로컬에서만 입력한다.

네임스페이스 적용
```bash
kubectl apply -f k8s/namespace/keycloak.yaml
```

관리자 Secret — 비밀번호는 로컬에서만 입력 (`$KEYCLOAK_ADMIN_PASSWORD`에 설정)
```bash
kubectl create secret generic keycloak-admin \
  -n keycloak \
  --from-literal=username=admin \
  --from-literal=password="$KEYCLOAK_ADMIN_PASSWORD"
```

Keycloak Helm repo 등록 (최초 1회)
```bash
helm repo add codecentric https://codecentric.github.io/helm-charts
helm repo update
```

Keycloakx 설치 (keycloak) — keycloak.nginx.lab.origemite.com
```bash
helm upgrade --install keycloak codecentric/keycloakx \
  -n keycloak \
  -f helm/values/keycloak.yaml \
  --wait \
  --timeout 15m
```
