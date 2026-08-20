# chaos ops

**kubectl 기반** 장애 주입 실습. Chaos Mesh 설치는 선택(아래 참고) — 랩은 `delete`/`scale`/`cordon`/`NetworkPolicy` 위주.

## 네임스페이스·대상

| 대상 | NS | 리소스 |
| --- | --- | --- |
| Gateway (nginx) | `nginx` | `deploy/nginx-ingress-nginx-controller` |
| Gateway (비교) | `ingress-compare` | `deploy/demo-echo` + 각 ingress/gateway |
| Redis | `redis` | `deploy/redis` |
| Kafka | `kafka` | `deploy/kafka` |
| mTLS 데모 | `ingress-compare`, `istio-system` | sidecar + [`mtls.md`](mtls.md) |
| NetworkPolicy | `cilium-policy` | [`cilium.md`](cilium.md) |

호스트·MetalLB: [`dns/inventory.yaml`](../../dns/inventory.yaml).

---

## 1) Pod delete (무작위/특정 파드)

단일 replica 워크로드 — 즉시 다운타임. PDB·multi-replica와 대비: [`pdb.md`](pdb.md).

```bash
kubectl -n redis delete pod -l app=redis --wait=false
kubectl -n redis get pods -w
kubectl -n redis rollout status deployment/redis
```

연속 삭제 (kubelet 재생성 관측):

```bash
while kubectl -n redis get pod -l app=redis -o name | head -1 | xargs -r kubectl -n redis delete --wait=false; do sleep 2; done
```

---

## 2) Node down

agent 중지·drain — 상세: [`node-failure.md`](node-failure.md).

```bash
NODE=k3d-lab-agent-0
kubectl cordon "$NODE"
kubectl drain "$NODE" --ignore-daemonsets --delete-emptydir-data --grace-period=30
```

또는:

```bash
docker stop k3d-lab-agent-0
kubectl get nodes
```

---

## 3) Gateway down (scale to zero)

기본 입구 nginx:

```bash
kubectl -n nginx scale deployment nginx-ingress-nginx-controller --replicas=0
curl -sS -o /dev/null -w "%{http_code}\n" -H 'Host: grafana.nginx.lab.origemite.com' http://172.18.255.201/
kubectl -n nginx scale deployment nginx-ingress-nginx-controller --replicas=1
kubectl -n nginx rollout status deployment/nginx-ingress-nginx-controller
```

Ingress 비교 데모:

```bash
kubectl -n ingress-compare scale deployment demo-echo --replicas=0
kubectl -n ingress-compare scale deployment demo-echo --replicas=1
```

---

## 4) Redis / Kafka down

```bash
kubectl -n redis scale deployment redis --replicas=0
kubectl -n kafka scale deployment kafka --replicas=0
```

복구:

```bash
kubectl -n redis scale deployment redis --replicas=1
kubectl -n kafka scale deployment kafka --replicas=1
kubectl -n redis rollout status deployment/redis
kubectl -n kafka rollout status deployment/kafka
```

연결 테스트 (다른 NS에서 debug pod):

```bash
kubectl run -it --rm redis-test --image=redis:7 --restart=Never -- \
  redis-cli -h redis.redis.svc ping
```

---

## 5) NetworkPolicy block (Cilium)

default-deny 후 client → echo 실패 — [`cilium.md`](cilium.md).

```bash
kubectl apply -f k8s/ciliumnetworkpolicy/default-deny-ingress.yaml
kubectl -n cilium-policy exec deploy/client -- wget -qO- --timeout=3 echo:80 || echo BLOCKED
cilium hubble observe --namespace cilium-policy --verdict Dropped
kubectl delete -f k8s/ciliumnetworkpolicy/default-deny-ingress.yaml
kubectl apply -f k8s/ciliumnetworkpolicy/allow-client-to-echo.yaml
```

Istio mTLS STRICT + 잘못된 PeerAuthentication으로 east-west 차단: [`mtls.md`](mtls.md).

---

## 검증 (metrics / logs / traces)

### Metrics — Prometheus / Grafana

```bash
kubectl -n monitoring port-forward svc/kube-prometheus-stack-prometheus 9090:9090
kubectl -n monitoring port-forward svc/kube-prometheus-stack-grafana 3000:80
```

예시 PromQL:

- `kube_deployment_status_replicas_available{namespace="redis"}`
- `kube_pod_container_status_restarts_total`
- `nginx_ingress_controller_requests`(nginx metrics 활성 시)

Ingress: `https://grafana.nginx.lab.origemite.com`

### Logs — Loki

Grafana Explore → Loki. nginx/redis/kafka namespace 필터:

```logql
{namespace="redis"} |= "error"
```

fluent-bit: [`helm/values/fluent-bit.yaml`](../../helm/values/fluent-bit.yaml)

### Traces — Tempo

장애 전후 동일 trace id·error span — [`otel-trace-lab.md`](otel-trace-lab.md).

```bash
kubectl -n tempo port-forward svc/tempo 4317:4317
```

Grafana Explore → Tempo, `status=error` 또는 latency spike.

### Mesh — Kiali / Hubble

```bash
kubectl -n kube-system port-forward svc/hubble-ui 12000:80
```

Kiali: `kiali.nginx.lab.origemite.com` — traffic graph에서 redis/kafka edge 소실 확인.

---

## 선택 — Chaos Mesh (미설치)

짧은 실험만 필요하면 위 kubectl로 충분. Mesh가 필요하면:

```bash
helm repo add chaos-mesh https://charts.chaos-mesh.org
helm upgrade --install chaos-mesh chaos-mesh/chaos-mesh \
  -n chaos-mesh --create-namespace \
  --set chaosDaemon.runtime=containerd \
  --set chaosDaemon.socketPath=/run/k3s/containerd/containerd.sock
```

PodChaos·NetworkChaos CR — **랩 필수 아님**. 설치 시 RBAC·k3d socket 경로 확인.

---

## 관련

- 시나리오: [`doc/scenarios/`](../../doc/scenarios/README.md)
- PDB: [`pdb.md`](pdb.md)
- Ingress 비교: [`ingress-compare.md`](ingress-compare.md)
- 관측: [`doc/06-observability.md`](../../doc/06-observability.md)
