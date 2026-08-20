# otel-trace-lab ops

분산 트레이스 실습 — **OTLP → Collector → Tempo → Grafana**. 앱 스텁(Spring Boot·Gateway·gRPC)은 선택; 먼저 파이프라인과 UI 접근을 익힌다.

## 아키텍처 (랩 기준)

```text
[Client]
   │ HTTP/gRPC
   ▼
[Gateway]  nginx / Istio / Kong … (ingress-compare 또는 nginx)
   │
   ▼
[Service]  (향후 Spring Boot stub — OTLP SDK)
   ├─► [Redis]   redis.redis.svc:6379
   └─► [Kafka]   kafka.kafka.svc:9092
         │
         │ span export (OTLP gRPC/HTTP)
         ▼
[OTel Collector]  otel-collector.otel.svc  :4317 / :4318
         │ otlp/tempo
         ▼
[Tempo]           tempo.tempo.svc:4317
         │ Grafana datasource
         ▼
[Grafana]         grafana.nginx.lab.origemite.com  (또는 port-forward)
```

계획된 span 종류:

| 구간 | 계측 | 비고 |
| --- | --- | --- |
| Gateway | ingress-nginx / Istio / Envoy | HTTP server span, W3C traceparent 전파 |
| Service | Spring Boot + Micrometer OTel | `server`, `redis`, `kafka` client span |
| Redis | Lettuce / Jedis instrumentation | `db.system=redis` |
| Kafka | Spring Kafka / OTel messaging | producer/consumer span |
| gRPC | OTel gRPC instrumentation | service-to-service |

Helm values:

- Collector: [`helm/values/opentelemetry-collector.yaml`](../../helm/values/opentelemetry-collector.yaml) — `traces` pipeline → `otlp/tempo` (`tempo.tempo.svc:4317`)
- Tempo: [`helm/values/tempo.yaml`](../../helm/values/tempo.yaml) — OTLP gRPC `:4317`, HTTP `:4318`
- Grafana Ingress: [`helm/values/kube-prometheus-stack.yaml`](../../helm/values/kube-prometheus-stack.yaml) — `grafana.nginx.lab.origemite.com`

## 사전조건

관측 스택 설치 — [`helm.md`](helm.md) Phase 2.

```bash
kubectl get ns monitoring tempo otel
kubectl get deploy -n otel otel-collector
kubectl get deploy -n tempo tempo
kubectl get ingress -n monitoring
```

Tempo datasource는 kube-prometheus-stack Grafana에 **수동 추가** 또는 values patch 예정. UI: **Explore → Tempo**, TraceQL 예 ` { .service.name = "demo-service" }`.

## Grafana / Tempo 접근 (복붙)

### Ingress (권장 — MetalLB·프록시 연결 후)

```bash
curl -sS -o /dev/null -w "%{http_code}\n" \
  -H 'Host: grafana.nginx.lab.origemite.com' \
  http://172.18.255.201/
```

브라우저: `https://grafana.nginx.lab.origemite.com` (TLS는 cert-manager·프록시 설정에 따름). DNS·프록시: [`proxy.md`](proxy.md), [`dns/inventory.yaml`](../../dns/inventory.yaml).

### port-forward (로컬만)

Grafana:

```bash
kubectl -n monitoring port-forward svc/kube-prometheus-stack-grafana 3000:80
```

브라우저: `http://127.0.0.1:3000` (기본 admin / chart secret — `kubectl -n monitoring get secret kube-prometheus-stack-grafana -o jsonpath='{.data.admin-password}' | base64 -d`).

Tempo OTLP (앱·curl 테스트용):

```bash
kubectl -n tempo port-forward svc/tempo 4317:4317 4318:4318
```

Collector OTLP (앱이 Collector 경유일 때):

```bash
kubectl -n otel port-forward svc/otel-collector 4317:4317 4318:4318
```

## Collector → Tempo 흐름 확인

Collector 로그:

```bash
kubectl -n otel logs deploy/otel-collector --tail=50 -f
```

Tempo 수신 확인 (테스트 span — port-forward 후):

```bash
curl -sS -X POST http://127.0.0.1:4318/v1/traces \
  -H 'Content-Type: application/json' \
  -d '{"resourceSpans":[{"resource":{"attributes":[{"key":"service.name","value":{"stringValue":"otel-trace-lab-manual"}}]},"scopeSpans":[{"spans":[{"traceId":"0123456789abcdef0123456789abcdef","spanId":"0123456789abcdef","name":"manual-test","kind":1,"startTimeUnixNano":"'$(date +%s)000000000'","endTimeUnixNano":"'$(date +%s)000000001'"}]}]}]}'
```

Grafana Explore → Tempo에서 `service.name=otel-trace-lab-manual` 검색.

## 랩 트레이스 경로 (목표 시나리오)

1. **요청** — `curl` 또는 브라우저 → `demo.nginx.lab.origemite.com` (또는 향후 Spring Boot `/api/orders`)
2. **Gateway** — nginx Ingress span (향후: access log + OTel nginx module 또는 mesh)
3. **Service** — 비즈니스 로직 span
4. **Redis** — cache get/set client span
5. **Kafka** — publish span → consumer span (별도 worker)
6. **수집** — 앱 `OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector.otel.svc:4317` (클러스터 내부) 또는 port-forward
7. **조회** — Grafana Tempo, 서비스 맵(Kiali 연계): [`istio.md`](istio.md), [`mtls.md`](mtls.md)

### 앱 환경 변수 (향후 Deployment stub)

```bash
OTEL_SERVICE_NAME=demo-service
OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector.otel.svc:4317
OTEL_EXPORTER_OTLP_PROTOCOL=grpc
OTEL_TRACES_EXPORTER=otlp
OTEL_METRICS_EXPORTER=otlp
OTEL_LOGS_EXPORTER=none
```

Redis/Kafka 엔드포인트 (클러스터 DNS):

```bash
REDIS_HOST=redis.redis.svc.cluster.local
KAFKA_BOOTSTRAP=kafka.kafka.svc.cluster.local:9092
```

## 메트릭·로그 연계

| 신호 | 저장 | 조회 |
| --- | --- | --- |
| Metrics | Collector → Prometheus remote write | Grafana → Prometheus |
| Logs | fluent-bit → Loki (values 별도) | Grafana → Loki |
| Traces | Collector → Tempo | Grafana → Tempo, Kiali |

카오스·장애 후 trace id로 로그 상관: Grafana **Trace to logs** (Loki datasource 연결 후).

## 선택 — 앱 스텁 Deployment

무겁지 않으면 `k8s/deployment/`에 echo + OTel auto-instrumentation Job 수준 stub 추가 가능. **필수 아님** — 위 manual OTLP POST와 ingress-compare 트래픽만으로도 파이프라인 검증 가능.

## 관련

- 튜토리얼: [`doc/06-observability.md`](../../doc/06-observability.md)
- Helm 설치: [`helm.md`](helm.md)
- Ingress 데모: [`ingress-compare.md`](ingress-compare.md)
- 데이터 워크로드: `k8s/deployment/redis.yaml`, `k8s/deployment/kafka.yaml`
- 카오스 후 trace 확인: [`chaos.md`](chaos.md)
