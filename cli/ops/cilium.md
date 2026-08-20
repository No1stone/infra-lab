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
