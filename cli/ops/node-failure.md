# node-failure ops

k3d **agent 노드** 장애, 유지보수 시뮬레이션. voluntary disruption은 [`pdb.md`](pdb.md), involuntary(노드 다운)는 여기.

## 사전조건

```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide | grep -v kube-system | head -30
```

1노드=1워크로드 라벨(`lab.origemite.com/workload=<이름>`) 매핑은 [`kube.md`](kube.md) 및 `k8s/node/*/labels.yaml` 참고.

## 시나리오 A — cordon + drain (계획 유지보수)

PDB 실습과 동일 패턴 — [`pdb.md`](pdb.md).

```bash
NODE=k3d-lab-agent-0
kubectl cordon "$NODE"
kubectl drain "$NODE" --ignore-daemonsets --delete-emptydir-data --grace-period=30 --timeout=300s
kubectl get pods -A -o wide | grep "$NODE"
kubectl uncordon "$NODE"
```

## 시나리오 B — k3d agent 중지 (노드 다운)

Ubuntu 노트북에서 Docker 컨테이너로 agent 표현.

```bash
docker ps --filter name=k3d-lab-agent --format '{{.Names}}'
docker stop k3d-lab-agent-0
kubectl get nodes
kubectl get pods -A --field-selector spec.nodeName=k3d-lab-agent-0
```

기대: Node `NotReady`, 해당 노드 파드 `Unknown`/`Terminating` → 스케줄러가 다른 agent로 재스케줄(replica>1, PDB, nodeSelector에 따름).

복구:

```bash
docker start k3d-lab-agent-0
kubectl get nodes -w
```

## 시나리오 C — k3d server 중지 (클러스터 API 불안)

**주의**: control plane 단일 server — lab 전체 API 중단. DR, 클러스터 업그레이드 시나리오용.

```bash
docker stop k3d-lab-server-0
kubectl get nodes
docker start k3d-lab-server-0
k3d cluster start lab --wait --timeout 120s
```

## 검증

```bash
kubectl get events -A --sort-by='.lastTimestamp' | tail -30
kubectl -n monitoring port-forward svc/kube-prometheus-stack-prometheus 9090:9090
```

Prometheus: `kube_node_status_condition{condition="Ready",status="true"}`, `kube_pod_status_ready`.

트레이스: 노드 이벤트 전후 요청 span — [`otel-trace-lab.md`](otel-trace-lab.md).

## 관련

- 카오스 종합: [`chaos.md`](chaos.md)
- 시나리오: [`doc/scenarios/02-node-upgrade.md`](../../doc/scenarios/02-node-upgrade.md), [`doc/scenarios/07-network-failure.md`](../../doc/scenarios/07-network-failure.md)
