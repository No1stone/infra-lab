# k3d ops

lab 클러스터 생성 — 서버 1, 에이전트 5, 80/443 로드밸런서, 준비 대기
```bash
k3d cluster create lab \
  --servers 1 \
  --agents 5 \
  --port 80:80@loadbalancer \
  --port 443:443@loadbalancer \
  --k3s-arg "--disable=traefik@server:*" \
  --wait \
  --timeout 120s
```

MetalLB 사용 시 ServiceLB(klipper) 끄고 생성
```bash
k3d cluster create lab \
  --servers 1 \
  --agents 5 \
  --port 80:80@loadbalancer \
  --port 443:443@loadbalancer \
  --k3s-arg "--disable=traefik@server:*" \
  --k3s-arg "--disable=servicelb@server:*" \
  --wait \
  --timeout 120s
```

Cilium 사용 시 k3s NetworkPolicy 끄고 생성 — 생성 후 cilium ops로 CNI 부트스트랩
```bash
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
```

lab 클러스터 목록 확인
```bash
k3d cluster list
```

lab 클러스터 시작
```bash
k3d cluster start lab --wait --timeout 120s
```

lab 클러스터 중지
```bash
k3d cluster stop lab
```

lab 클러스터 삭제
```bash
k3d cluster delete lab
```

lab kubeconfig를 기본 파일에 병합하고 컨텍스트 전환
```bash
k3d kubeconfig merge lab --kubeconfig-merge-default --kubeconfig-switch-context
```

에이전트 노드 라벨은 생성 후 kubectl로 붙인다 — kube ops 참고
```bash
kubectl get nodes -o wide
```
