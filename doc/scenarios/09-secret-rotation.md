# 09 Secret 로테이션

## 목표

애플리케이션·Ingress·GitOps가 참조하는 **Kubernetes Secret** 값을 바꾸고, Pod reload(rolling restart) 없이/와 함께 **무중단** 반영 방법을 연습한다.

## 사전조건

- 대상 NS 예: `argocd`, `keycloak`, `harbor` — [`cli/ops/keycloak.md`](../../cli/ops/keycloak.md), [`cli/ops/harbor.md`](../../cli/ops/harbor.md)
- (선택) Grafana admin secret — `kube-prometheus-stack-grafana`

## 절차

1. **Secret inventory**
   ```bash
   kubectl get secrets -n argocd
   kubectl get secrets -n keycloak
   ```
2. **값 로테이션 (예: DB password)** — 새 Secret 버전 생성
   ```bash
   kubectl -n keycloak create secret generic keycloak-db \
     --from-literal=password='NEW_PLACEHOLDER' \
     --dry-run=client -o yaml | kubectl apply -f -
   ```
   **실값은 커밋하지 않음** — 랩에서는 placeholder
3. **Deployment envFrom / volume mount** — Secret 변경만으로 Pod에 **자동 반영 안 됨** → rolling restart
   ```bash
   kubectl -n keycloak rollout restart deployment/keycloak
   kubectl -n keycloak rollout status deployment/keycloak
   ```
4. **TLS Secret** — cert-manager 갱신과 연동 — [`doc/scenarios/08-certificate-renewal.md`](08-certificate-renewal.md)
5. **Harbor / registry pull secret** — robot account rotate → `imagePullSecrets` 갱신 — [`cli/ops/harbor.md`](../../cli/ops/harbor.md)
6. **(계획) External Secrets Operator** — Vault/AWS SM → K8s Secret sync

## 검증

- 앱 login / API 401→200 (Keycloak admin)
- `kubectl -n keycloak get pods` — restart 후 Ready
- Argo CD: repo credential 변경 시 sync — [`cli/ops/argocd.md`](../../cli/ops/argocd.md)
- audit: `kubectl get events -n keycloak`

## 관련

- [`doc/scenarios/08-certificate-renewal.md`](08-certificate-renewal.md)
- [`doc/13-keycloak.md`](../13-keycloak.md)
- [`k8s/ingress/vault.yaml`](../../k8s/ingress/vault.yaml) (Vault 연계 계획)
- [`cli/doc/kube.md`](../../cli/doc/kube.md) secret 절
