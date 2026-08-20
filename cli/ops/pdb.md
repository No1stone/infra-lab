# pdb ops

PodDisruptionBudget(PDB)로 **voluntary disruption**(drain, 노드 유지보수) 시에도 최소 가용 파드를 보장하는 실습. `ops-pdb` 네임스페이스, `nginx:1.27-alpine` 3 replica, `minAvailable: 2`.

## 매니페스트 적용

```bash
kubectl apply -f k8s/namespace/ops-pdb.yaml
kubectl apply -f k8s/deployment/ops-pdb-web.yaml
kubectl apply -f k8s/service/ops-pdb-web.yaml
kubectl apply -f k8s/poddisruptionbudget/ops-pdb-web.yaml
kubectl -n ops-pdb rollout status deployment/ops-pdb-web
kubectl -n ops-pdb get pods -o wide
kubectl -n ops-pdb get pdb
```

## 연속 요청 (백그라운드)

포트 포워드 후 curl 루프로 다운타임 관측. 터미널 A에서 실행하고 drain 실습은 터미널 B에서 진행.

```bash
kubectl -n ops-pdb port-forward svc/ops-pdb-web 8080:80
```

```bash
while true; do
 curl -s -o /dev/null -w "%{http_code} %{time_total}s\n" http://127.0.0.1:8080/ || echo "fail"
 sleep 0.5
done
```

백그라운드로 돌릴 때:

```bash
while true; do curl -s -o /dev/null -w "%{http_code}\n" http://127.0.0.1:8080/ || echo fail; sleep 0.5; done > /tmp/ops-pdb-curl.log 2>&1 &
echo $! > /tmp/ops-pdb-curl.pid
tail -f /tmp/ops-pdb-curl.log
```

종료:

```bash
kill "$(cat /tmp/ops-pdb-curl.pid)"
```

## drain — PDB가 eviction을 막는지 확인

drain 대상 노드를 고른다. `ops-pdb-web` 파드가 올라간 agent 노드 1개를 사용.

```bash
kubectl get nodes -o wide
kubectl -n ops-pdb get pods -o wide
NODE=k3d-lab-agent-0
kubectl cordon "$NODE"
kubectl drain "$NODE" --ignore-daemonsets --delete-emptydir-data --grace-period=30
```

기대: PDB `minAvailable: 2` 때문에 한 번에 2개 이상 파드 evict 불가. drain이 **Blocked** 또는 **Waiting** 상태로 멈춤.

```bash
kubectl -n ops-pdb get pdb ops-pdb-web -o yaml
kubectl get events -A --field-selector involvedObject.kind=Pod --sort-by='.lastTimestamp' | tail -20
```

drain 중단, 노드 복구:

```bash
kubectl uncordon "$NODE"
```

## PDB 삭제 후 drain 비교

PDB 없으면 drain이 연속 evict 가능. curl 로그에서 502/연결 실패가 늘 수 있음.

```bash
kubectl delete -f k8s/poddisruptionbudget/ops-pdb-web.yaml
kubectl drain "$NODE" --ignore-daemonsets --delete-emptydir-data --grace-period=30
```

비교 후 PDB 복원:

```bash
kubectl apply -f k8s/poddisruptionbudget/ops-pdb-web.yaml
kubectl uncordon "$NODE"
```

## 비교: PDB vs Deployment rollingUpdate

| 항목 | 역할 | 이 실습 값 |
| --- | --- | --- |
| PDB `minAvailable` | voluntary disruption 시 **동시에 없어질 수 있는** 파드 상한 | `2` → 3 replica 중 최소 2개 유지 |
| PDB `maxUnavailable` | `minAvailable` 대신 “최대 N개 unavailable” 표현 (둘 중 하나만) | — |
| Deployment `maxUnavailable` | **rolling update** 시 old+new 동시 unavailable 허용 | `0` → 업데이트 중에도 serving 유지 |
| Deployment `maxSurge` | rolling update 시 extra 파드 허용 | `1` → 최대 4개까지 일시 증가 |

- **PDB**: kubelet eviction, drain, cluster-autoscaler scale-down 등 *클러스터가 파드를 내쫓을 때* 적용.
- **Deployment strategy**: `kubectl rollout` / 이미지 변경 등 *앱 업데이트* 시 적용.
- 둘 다 503 없이 서비스하려면 probe(readiness) + PDB + `maxUnavailable: 0` 조합이 일반적.

PDB를 `maxUnavailable: 1`로 바꿔 비교:

```bash
kubectl -n ops-pdb patch pdb ops-pdb-web --type merge -p '{"spec":{"minAvailable":null,"maxUnavailable":1}}'
kubectl -n ops-pdb get pdb ops-pdb-web
```

원복:

```bash
kubectl apply -f k8s/poddisruptionbudget/ops-pdb-web.yaml
```

## 정리

```bash
kubectl delete namespace ops-pdb
```
