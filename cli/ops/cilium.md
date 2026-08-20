# cilium ops

Cilium은 k3s 기본 CNI·NetworkPolicy 대신 쓴다. **클러스터 생성 시** `--disable-network-policy`가 필요하므로 lab를 삭제·재생성한 뒤 설치한다.

k3d mount 이슈 회피 (Ubuntu 노트북에서 cilium 설치 전)
```bash
export K3D_FIX_MOUNTS=1
```

lab 클러스터 삭제·재생성 — k3d ops의 Cilium용 create 블록 사용
```bash
k3d cluster delete lab
k3d cluster create lab \
  --servers 1 \
  --agents 5 \
  --port 80:80@loadbalancer \
  --port 443:443@loadbalancer \
  --k3s-arg "--disable=traefik@server:*" \
  --k3s-arg "--disable=servicelb@server:*" \
  --k3s-arg "--disable-network-policy@server:*" \
  --wait \
  --timeout 120s
k3d kubeconfig merge lab --kubeconfig-merge-default --kubeconfig-switch-context
```

Cilium Helm 설치 (kube-system)
```bash
helm repo add cilium https://helm.cilium.io/
helm repo update
helm upgrade --install cilium cilium/cilium \
  -n kube-system \
  -f helm/values/cilium.yaml \
  --wait \
  --timeout 10m
```

Cilium 상태 확인 (CLI 설치된 경우)
```bash
cilium status --wait
cilium connectivity test
```

## Hubble UI

`helm/values/cilium.yaml`에서 Hubble relay·UI 활성. 포트 포워드 후 브라우저에서 흐름 확인.

```bash
kubectl -n kube-system port-forward svc/hubble-ui 12000:80
```

Hubble CLI 관측 예 (relay 경유):

```bash
cilium hubble port-forward &
hubble observe --namespace cilium-policy
hubble observe --namespace cilium-policy --pod echo --protocol tcp
hubble observe --verdict Dropped --namespace cilium-policy
```

## 파드간 권한 데모 (CiliumNetworkPolicy)

네임스페이스·워크로드·서비스 적용:

```bash
kubectl apply -f k8s/namespace/cilium-policy.yaml
kubectl apply -f k8s/deployment/cilium-policy-echo.yaml
kubectl apply -f k8s/deployment/cilium-policy-client.yaml
kubectl apply -f k8s/service/cilium-policy-echo.yaml
kubectl -n cilium-policy rollout status deployment/echo deployment/client
```

정책 없을 때 client → echo (성공):

```bash
kubectl -n cilium-policy exec deploy/client -- wget -qO- echo:80
```

default-deny ingress 적용 후 (실패):

```bash
kubectl apply -f k8s/ciliumnetworkpolicy/default-deny-ingress.yaml
kubectl -n cilium-policy exec deploy/client -- wget -qO- --timeout=3 echo:80
```

client → echo 허용 정책 적용 후 (성공):

```bash
kubectl apply -f k8s/ciliumnetworkpolicy/allow-client-to-echo.yaml
kubectl -n cilium-policy exec deploy/client -- wget -qO- echo:80
```

권장 순서: ns+deploy+svc → 연결 확인 → `default-deny-ingress` → 차단 확인 → `allow-client-to-echo` → client만 허용 확인. Hubble에서 `verdict:Dropped`/`Forwarded`로 대조.

정책 제거:

```bash
kubectl delete -f k8s/ciliumnetworkpolicy/allow-client-to-echo.yaml
kubectl delete -f k8s/ciliumnetworkpolicy/default-deny-ingress.yaml
```
