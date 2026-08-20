# mtls ops

Istio mesh **east-west** mTLS. AWS 프록시(`*.lab.origemite.com`) TLS와는 별개다.

## 사전 조건

Istio 설치 완료 — [`cli/ops/istio.md`](istio.md). 데모 NS는 `bookinfo` 또는 Phase 7 `ingress-compare` 중 하나.

```bash
kubectl get ns istio-system bookinfo ingress-compare 2>/dev/null
kubectl get deploy -n istio-system istiod
```

## sidecar 자동 주입

`ingress-compare`(Phase 7) 또는 `bookinfo`에 injection 라벨.

```bash
kubectl label namespace ingress-compare istio-injection=enabled --overwrite
```

bookinfo 사용 시:

```bash
kubectl create namespace bookinfo --dry-run=client -o yaml | kubectl apply -f -
kubectl label namespace bookinfo istio-injection=enabled --overwrite
```

기존 Pod는 재시작해야 sidecar가 붙는다.

```bash
kubectl rollout restart deployment -n ingress-compare
```

## PeerAuthentication STRICT

저장소 매니페스트 적용(권장):

```bash
kubectl apply -f k8s/peerauthentication/istio-mtls.yaml
```

HEREDOC으로 `ingress-compare`만 STRICT:

```bash
kubectl apply -f - <<'EOF'
apiVersion: security.istio.io/v1
kind: PeerAuthentication
metadata:
  name: default
  namespace: ingress-compare
spec:
  mtls:
    mode: STRICT
EOF
```

mesh 전체 STRICT는 `istio-system`에 동일 kind·`mode: STRICT`로 `metadata.namespace: istio-system` — `k8s/peerauthentication/istio-mtls.yaml` 주석 참고.

## DestinationRule (선택)

기본 mesh mTLS는 istiod·PeerAuthentication으로 충분한 경우가 많다. 특정 서비스 subset·TLS mode를 명시할 때만 `DestinationRule`의 `trafficPolicy.tls.mode: ISTIO_MUTUAL`을 추가한다.

## 검증

```bash
kubectl get peerauthentication -A
kubectl get pods -n ingress-compare -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[*].name}{"\n"}{end}'
```

`istioctl` 사용 가능 시:

```bash
istioctl authn tls-check demo-echo.ingress-compare.svc.cluster.local ingress-compare
```

기대: PeerAuthentication `STRICT`, Pod에 `istio-proxy` 컨테이너 존재.
