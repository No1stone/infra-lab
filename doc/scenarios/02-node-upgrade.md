# 02 노드 업그레이드

## 목표

agent 노드 **유지보수(drain)** 시 워크로드가 다른 노드로 재스케줄되고, PDB로 **voluntary disruption** 중에도 최소 가용성을 유지한다. k3d에서 agent 중지로 **involuntary** 장애도 비교한다.

## 사전조건

- k3d `lab`: server 1 + agent 5 — [`cli/ops/k3d.md`](../../cli/ops/k3d.md)
- 노드 라벨: `k8s/node/*/labels.yaml` (1노드=1워크로드)
- PDB 실습 NS: `ops-pdb` — [`cli/ops/pdb.md`](../../cli/ops/pdb.md)

## 절차

1. **노드, 파드 배치 확인**
 ```bash
 kubectl get nodes -o wide
 kubectl -n ops-pdb get pods -o wide
 ```
2. **연속 트래픽** — [`cli/ops/pdb.md`](../../cli/ops/pdb.md) curl 루프
3. **cordon + drain** (계획 유지보수)
 ```bash
 NODE=k3d-lab-agent-0
 kubectl cordon "$NODE"
 kubectl drain "$NODE" --ignore-daemonsets --delete-emptydir-data --grace-period=30
 ```
4. **PDB blocked 관측** — drain이 PDB `minAvailable: 2`에 막히는지 events 확인
5. **노드 복구** — `uncordon`, drain 완료 또는 중단
6. **(비교) agent docker stop** — [`cli/ops/node-failure.md`](../../cli/ops/node-failure.md) 시나리오 B
7. **데이터 워크로드** — `redis`/`kafka` single replica + nodeSelector: drain 시 **다른 agent에 스케줄 불가**면 Pending — 라벨, replica 정책 점검

## 검증

- curl: drain 중 502/503 패턴 (PDB 유무, replica 수에 따라)
- `kubectl get events -A | tail -30`
- `kubectl -n ops-pdb get pdb`
- Prometheus: `kube_node_status_condition`, pod ready time
- Hubble: drop/재스케줄 구간 — [`cli/ops/cilium.md`](../../cli/ops/cilium.md)

## 관련

- [`cli/ops/node-failure.md`](../../cli/ops/node-failure.md)
- [`cli/ops/pdb.md`](../../cli/ops/pdb.md)
- [`cli/ops/chaos.md`](../../cli/ops/chaos.md) § Node down
- [`doc/scenarios/01-zero-downtime-update.md`](01-zero-downtime-update.md)
