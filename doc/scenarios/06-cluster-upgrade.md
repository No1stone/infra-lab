# 06 클러스터 업그레이드

## 목표

k3d/k3s **패치·마이너 업그레이드** 순서를 익힌다: control plane → CRD/Helm → data plane(Cilium/Istio) → 워크로드. 업그레이드 중 API 가용성·Pod disruption을 관측한다.

## 사전조건

- 동작 중인 `lab` 클러스터
- Helm release 목록·values Git 버전 고정
- PDB 실습 NS (선택) — drain과 병행

## 절차

1. **사전 점검**
   ```bash
   kubectl version
   k3d version
   helm list -A
   cilium version 2>/dev/null || true
   ```
2. **워크로드 스냅샷** — `kubectl get pods -A -o wide`, 중요 Deployment replica 확인
3. **k3d 이미지 업그레이드** — k3d cluster stop → `k3d cluster start` with newer k3s image (k3d 문서 기준)
   ```bash
   k3d cluster stop lab
   # k3d cluster create ... --image rancher/k3s:v1.xx.x-k3s1  재생성 또는 k3d upgrade 절차
   k3d cluster start lab --wait --timeout 300s
   ```
   랩에서는 **재생성(DR)** 이 현실적 — [`doc/scenarios/05-disaster-recovery.md`](05-disaster-recovery.md)
4. **CRD / Gateway API** — [`cli/ops/ingress-compare.md`](../../cli/ops/ingress-compare.md) CRD 버전 확인
5. **Cilium** — `helm upgrade cilium` — [`cli/ops/cilium.md`](../../cli/ops/cilium.md), connectivity test
6. **Istio** — istiod → gateway 순 — [`cli/ops/istio.md`](../../cli/ops/istio.md)
7. **kube-prometheus-stack / cert-manager** — chart appVersion 호환 matrix 확인 후 `helm upgrade`
8. **노드 순차 drain** — 업그레이드 후 kubelet 재시작 검증 — [`doc/scenarios/02-node-upgrade.md`](02-node-upgrade.md)

## 검증

- `kubectl get nodes` Ready, 버전 일치
- `cilium connectivity test` (설치 시)
- `demo.*.lab.origemite.com` 7종 curl — [`cli/ops/ingress-compare.md`](../../cli/ops/ingress-compare.md)
- Argo CD sync healthy — [`cli/ops/argocd.md`](../../cli/ops/argocd.md)

## 관련

- [`cli/ops/k3d.md`](../../cli/ops/k3d.md)
- [`cli/ops/helm.md`](../../cli/ops/helm.md)
- [`doc/03-terraform.md`](../03-terraform.md) (향후 IaC 버전 pin)
- [`doc/scenarios/02-node-upgrade.md`](02-node-upgrade.md)
