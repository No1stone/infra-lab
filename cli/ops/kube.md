# kube ops

lab 에이전트 노드에 워크로드 라벨 부여 — agent-0~4 → mysql, redis, kafka, rabbitmq, vault
```bash
kubectl label node k3d-lab-agent-0 lab.origemite.com/workload=mysql --overwrite
kubectl label node k3d-lab-agent-1 lab.origemite.com/workload=redis --overwrite
kubectl label node k3d-lab-agent-2 lab.origemite.com/workload=kafka --overwrite
kubectl label node k3d-lab-agent-3 lab.origemite.com/workload=rabbitmq --overwrite
kubectl label node k3d-lab-agent-4 lab.origemite.com/workload=vault --overwrite
```

노드 라벨 확인
```bash
kubectl get nodes -L lab.origemite.com/workload
```

네임스페이스 적용
```bash
kubectl apply -f k8s/namespace/
```

ConfigMap 적용 (MetalLB IP 풀, ClusterIssuer, 워크로드 설정)
```bash
kubectl apply -f k8s/configmap/
```

워크로드 노드 라벨 ConfigMap 적용 (mysql, redis, kafka, rabbitmq, vault)
```bash
kubectl apply -f k8s/node/mysql/labels.yaml
kubectl apply -f k8s/node/redis/labels.yaml
kubectl apply -f k8s/node/kafka/labels.yaml
kubectl apply -f k8s/node/rabbitmq/labels.yaml
kubectl apply -f k8s/node/vault/labels.yaml
```

Deployment, Service, Ingress 적용
```bash
kubectl apply -f k8s/deployment/
kubectl apply -f k8s/service/
kubectl apply -f k8s/ingress/
```

lab 클러스터 파드 스냅샷 (전체 네임스페이스)
```bash
kubectl get pods -A -o wide
```

lab 클러스터 Service 스냅샷 (전체 네임스페이스)
```bash
kubectl get svc -A
```

lab 클러스터 Ingress 스냅샷 (전체 네임스페이스)
```bash
kubectl get ingress -A
```

워크로드 파드 상태 확인 (mysql, redis, kafka, rabbitmq, vault)
```bash
kubectl get pods -n mysql -o wide
kubectl get pods -n redis -o wide
kubectl get pods -n kafka -o wide
kubectl get pods -n rabbitmq -o wide
kubectl get pods -n vault -o wide
```
