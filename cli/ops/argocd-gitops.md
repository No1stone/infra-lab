# argocd-gitops ops

`demo-echo` Application으로 GitOps·drift·Self Heal 실습. Argo CD UI: `argocd.nginx.lab.origemite.com`.

## 사전조건

- k3d 클러스터, `kubectl` 컨텍스트 연결
- MetalLB + nginx Ingress (Phase 1)
- Application `repoURL`을 실제 org/호스트로 바꾼 뒤 진행

## 1) Argo CD 설치 확인

```bash
kubectl -n argocd get pods
kubectl -n argocd get ingress
```

없으면 Helm으로 설치 (`helm.md` Phase 3).

```bash
helm upgrade --install argocd argo/argo-cd \
  -n argocd \
  --create-namespace \
  -f helm/values/argocd.yaml \
  --wait \
  --timeout 10m
```

공개 Git 저장소 등록 (비공개면 `--username` / `--password` 또는 SSH 키 추가).

```bash
argocd repo add https://github.com/<org>/infra-lab.git \
  --server argocd.nginx.lab.origemite.com --grpc-web --insecure
```

## 2) Application 적용

`argocd/application/demo-echo.yaml`의 `repoURL`을 실제 값으로 수정한 뒤:

```bash
kubectl apply -f argocd/application/demo-echo.yaml
```

상태 확인.

```bash
argocd app get demo-echo \
  --server argocd.nginx.lab.origemite.com --grpc-web --insecure
kubectl -n ingress-compare get deploy,svc
```

## 3) drift 만들기 (클러스터 수동 변경)

Git은 그대로 두고 live state만 바꾼다.

```bash
kubectl -n ingress-compare edit deployment demo-echo
# spec.replicas: 3 또는 args "-text=drift-test" 로 변경 후 저장
```

또는 one-liner:

```bash
kubectl -n ingress-compare patch deployment demo-echo \
  -p '{"spec":{"replicas":3}}'
```

Argo CD는 OutOfSync로 표시된다.

```bash
argocd app get demo-echo \
  --server argocd.nginx.lab.origemite.com --grpc-web --insecure
argocd app diff demo-echo \
  --server argocd.nginx.lab.origemite.com --grpc-web --insecure
```

## 4) Self Heal — Git desired state로 복원

`syncPolicy.automated.selfHeal: true`이면 수 초~수십 초 내 자동 복원된다. 수동으로도 가능:

```bash
argocd app sync demo-echo \
  --server argocd.nginx.lab.origemite.com --grpc-web --insecure
kubectl -n ingress-compare get deployment demo-echo -o jsonpath='{.spec.replicas}{"\n"}'
```

기대: replicas `1`, echo 텍스트 `ingress-compare-demo`.

## 5) Git 변경 시뮬레이션 (Auto Sync)

로컬에서 매니페스트를 고치고 push하면 Auto Sync가 클러스터에 반영한다.

```bash
# 예: k8s/deployment/demo-echo.yaml 의 args 텍스트 변경 → commit → push
argocd app get demo-echo \
  --server argocd.nginx.lab.origemite.com --grpc-web --insecure
kubectl -n ingress-compare get deployment demo-echo -o yaml | grep -A2 'args:'
```

변경 되돌릴 때는 Git에서 revert 후 push.

## 6) Prune 실습 (선택)

Application이 관리하던 리소스를 Git에서 제거하고 push하면 `prune: true`가 클러스터에서도 삭제한다. 실습 후 Git을 원복할 것.

## 용어 (짧게)

| 항목 | 의미 |
| --- | --- |
| **Drift** | Git(desired)과 클러스터(live) 불일치. 수동 `kubectl edit`·장애 복구 중 생김 |
| **Auto Sync** | Git 변경을 주기적으로 감지해 클러스터에 자동 적용 |
| **Self Heal** | drift 발생 시 Git 기준으로 live state를 되돌림 |
| **Prune** | Git에 없는 리소스를 클러스터에서 삭제 |

## 정리

```bash
argocd app delete demo-echo --yes \
  --server argocd.nginx.lab.origemite.com --grpc-web --insecure
```

## 관련

- Application: [`argocd/application/demo-echo.yaml`](../../argocd/application/demo-echo.yaml)
- Helm 설치: [`helm.md`](helm.md) Phase 3
- CLI 레퍼런스: [`../doc/argocd.md`](../doc/argocd.md)
