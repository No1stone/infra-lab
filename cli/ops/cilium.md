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

## Hubble UI

`helm/values/cilium.yaml`에서 Hubble relay·UI 활성. 포트 포워드 후 브라우저에서 흐름 확인.

```bash
kubectl -n kube-system port-forward svc/hubble-ui 12000:80
```

Hubble CLI 관측 예 (relay 경유):

```bash
cilium hubble port-forward &
hubble observe --namespace cilium-policy
hubble observe --namespace cilium-policy --pod echo --protocol tcp
hubble observe --verdict Dropped --namespace cilium-policy
```

## 파드간 권한 데모 (CiliumNetworkPolicy)

네임스페이스·워크로드·서비스 적용:

```bash
kubectl apply -f k8s/namespace/cilium-policy.yaml
kubectl apply -f k8s/deployment/cilium-policy-echo.yaml
kubectl apply -f k8s/deployment/cilium-policy-client.yaml
kubectl apply -f k8s/service/cilium-policy-echo.yaml
kubectl -n cilium-policy rollout status deployment/echo deployment/client
```

정책 없을 때 client → echo (성공):

```bash
kubectl -n cilium-policy exec deploy/client -- wget -qO- echo:80
```

default-deny ingress 적용 후 (실패):

```bash
kubectl apply -f k8s/ciliumnetworkpolicy/default-deny-ingress.yaml
kubectl -n cilium-policy exec deploy/client -- wget -qO- --timeout=3 echo:80
```

client → echo 허용 정책 적용 후 (성공):

```bash
kubectl apply -f k8s/ciliumnetworkpolicy/allow-client-to-echo.yaml
kubectl -n cilium-policy exec deploy/client -- wget -qO- echo:80
```

권장 순서: ns+deploy+svc → 연결 확인 → `default-deny-ingress` → 차단 확인 → `allow-client-to-echo` → client만 허용 확인. Hubble에서 `verdict:Dropped`/`Forwarded`로 대조.

정책 제거:

```bash
kubectl delete -f k8s/ciliumnetworkpolicy/allow-client-to-echo.yaml
kubectl delete -f k8s/ciliumnetworkpolicy/default-deny-ingress.yaml
```

## L3/L4 정책 복습

`CiliumNetworkPolicy`는 **선택된 Pod(endpointSelector)** 에 ingress/egress 규칙을 붙인다. Kubernetes `NetworkPolicy`와 문법이 비슷하지만 Cilium 확장(L7·FQDN)을 쓸 수 있다.

| 단계 | 파일 | 동작 |
| --- | --- | --- |
| 1 | (정책 없음) | NS 내 기본 allow — client → echo OK |
| 2 | `default-deny-ingress.yaml` | `ingress: []` — echo 포함 NS 전체 ingress 차단 |
| 3 | `allow-client-to-echo.yaml` | `app:client` → `app:echo` TCP 80·8080만 허용 |

L3/L4만 볼 때 핵심 필드:

- `endpointSelector` — 정책이 적용되는 **대상 Pod** (예: echo)
- `ingress[].fromEndpoints` — **누가** 접속 가능한지 (예: client)
- `toPorts[].ports` — **어떤 포트·프로토콜** (Pod containerPort 기준 8080)

정책 목록·요약:

```bash
kubectl -n cilium-policy get ciliumnetworkpolicies
cilium policy get -n cilium-policy 2>/dev/null || true
```

## L7 HTTP 정책 (GET / 만 허용)

agnhost `netexec` echo는 **8080**에서 HTTP. L7 규칙은 **containerPort**에 맞춘다.

사전: Cilium Helm에 L7 proxy 활성 필요(미설정 시 L4만 동작). [`helm/values/cilium.yaml`](../helm/values/cilium.yaml)에 `l7Proxy` 등 버전별 키 확인.

권장 순서: ns+deploy+svc → L4 allow → L7 적용 → GET/POST 비교.

```bash
kubectl apply -f k8s/ciliumnetworkpolicy/l7-http-get-only.yaml
kubectl -n cilium-policy exec deploy/client -- wget -qO- echo:80/
kubectl -n cilium-policy exec deploy/client -- wget -qO- --post-data=x echo:80/ || true
```

기대: GET `/` 성공, POST 등 비-GET 또는 다른 path는 차단(L7 proxy ON 시).

제거:

```bash
kubectl delete -f k8s/ciliumnetworkpolicy/l7-http-get-only.yaml
```

매니페스트: [`k8s/ciliumnetworkpolicy/l7-http-get-only.yaml`](../../k8s/ciliumnetworkpolicy/l7-http-get-only.yaml)

## DNS / FQDN egress 정책

외부 HTTPS를 **FQDN 화이트리스트**로만 허용하는 예. DNS proxy·`toFQDNs` 지원이 Cilium values에 켜져 있어야 한다.

```bash
kubectl apply -f k8s/ciliumnetworkpolicy/fqdn-allow-example.yaml
kubectl -n cilium-policy exec deploy/client -- wget -qO- --timeout=5 https://kubernetes.io/ || true
kubectl -n cilium-policy exec deploy/client -- wget -qO- --timeout=5 https://example.com/ || true
```

기대(정책·외부 egress OK 시): `kubernetes.io` 허용, `example.com` 차단. 클러스터가 외부망 없으면 스킵.

제거:

```bash
kubectl delete -f k8s/ciliumnetworkpolicy/fqdn-allow-example.yaml
```

매니페스트: [`k8s/ciliumnetworkpolicy/fqdn-allow-example.yaml`](../../k8s/ciliumnetworkpolicy/fqdn-allow-example.yaml)

## Hubble — Drop / Forward / observe 분석

relay 포워드(백그라운드):

```bash
cilium hubble port-forward &
sleep 2
```

**Forwarded** (허용된 흐름):

```bash
hubble observe --namespace cilium-policy --verdict FORWARDED --protocol tcp
hubble observe --namespace cilium-policy --pod echo --verdict FORWARDED
```

**Dropped** (정책 차단):

```bash
hubble observe --namespace cilium-policy --verdict DROPPED
hubble observe --namespace cilium-policy --from-pod client --verdict DROPPED
```

L7 HTTP( L7 proxy·Hubble L7 ON 시):

```bash
hubble observe --namespace cilium-policy --protocol http
hubble observe --namespace cilium-policy --http-method GET --verdict FORWARDED
```

정책 전후 diff — default-deny 적용 직후 client wget 한 번, Dropped 카운트 확인:

```bash
kubectl apply -f k8s/ciliumnetworkpolicy/default-deny-ingress.yaml
kubectl -n cilium-policy exec deploy/client -- wget -qO- --timeout=3 echo:80 || true
hubble observe --namespace cilium-policy --last 20 --verdict DROPPED
```

allow 복구 후 Forwarded:

```bash
kubectl apply -f k8s/ciliumnetworkpolicy/allow-client-to-echo.yaml
kubectl -n cilium-policy exec deploy/client -- wget -qO- echo:80
hubble observe --namespace cilium-policy --last 10 --verdict FORWARDED --from-pod client
```

UI: `kubectl -n kube-system port-forward svc/hubble-ui 12000:80` → Flow filter `verdict:DROPPED` / `namespace:cilium-policy`.
