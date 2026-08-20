# metallb ops

metallb Helm 설치 후 IP 풀, L2Advertisement 적용 — apply 전 k8s/configmap/metallb-ip-pool.yaml의 addresses를 k3d Docker 네트워크 CIDR에 맞게 수정
```bash
docker network inspect k3d-lab -f '{{range .IPAM.Config}}{{.Subnet}}{{end}}'
kubectl apply -f k8s/configmap/metallb-ip-pool.yaml
```
