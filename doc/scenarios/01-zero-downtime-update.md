# 01 무중단 롤링 업데이트

## 목표

Deployment **rolling update**로 이미지·설정을 바꿔도 사용자 요청에 **503/연결 끊김 없이** serving을 유지한다. readiness probe + `maxUnavailable: 0` + PDB 조합을 이해한다.

## 사전조건

- k3d `lab` 클러스터, kubectl 컨텍스트 `k3d-lab`
- [`cli/ops/pdb.md`](../../cli/ops/pdb.md) 매니페스트 적용 가능 (`ops-pdb` NS, 3 replica)
- (선택) Grafana/Prometheus — [`cli/ops/helm.md`](../../cli/ops/helm.md) Phase 2

## 절차

1. **워크로드 배포** — `ops-pdb-web` 3 replica + PDB `minAvailable: 2`
   - [`cli/ops/pdb.md`](../../cli/ops/pdb.md) 「매니페스트 적용」
2. **연속 요청** — port-forward + curl 루프로 HTTP 코드·latency 기록
3. **rolling update** — 이미지 태그 변경 (예: `nginx:1.27-alpine` → `1.28-alpine`)
   ```bash
   kubectl -n ops-pdb set image deployment/ops-pdb-web web=nginx:1.28-alpine
   kubectl -n ops-pdb rollout status deployment/ops-pdb-web --timeout=180s
   ```
4. **Deployment strategy 확인** — `maxSurge: 1`, `maxUnavailable: 0` (`k8s/deployment/ops-pdb-web.yaml`)
5. **PDB와 구분** — rolling update는 Deployment strategy; drain/eviction은 PDB — [`cli/ops/pdb.md`](../../cli/ops/pdb.md) 표 참고
6. **실패 주입(선택)** — readiness probe 실패 이미지로 503 구간 관측 후 `rollout undo`
   ```bash
   kubectl -n ops-pdb rollout undo deployment/ops-pdb-web
   ```

## 검증

- curl 로그: 업데이트 구간 **연속 200** (또는 허용 짧은 지연만)
- `kubectl -n ops-pdb get rs,pods -o wide` — old/new ReplicaSet 동시 존재·순차 교체
- Prometheus: `kube_deployment_status_replicas_available{deployment="ops-pdb-web"}`
- (선택) Tempo/Grafana — 업데이트 전후 trace latency — [`cli/ops/otel-trace-lab.md`](../../cli/ops/otel-trace-lab.md)

## 관련

- [`cli/ops/pdb.md`](../../cli/ops/pdb.md)
- [`k8s/deployment/ops-pdb-web.yaml`](../../k8s/deployment/ops-pdb-web.yaml)
- [`doc/scenarios/02-node-upgrade.md`](02-node-upgrade.md)
- [`cli/doc/kube.md`](../../cli/doc/kube.md) rollout 절
