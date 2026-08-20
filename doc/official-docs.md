# 공식 문서, 레퍼런스 URL

infra-lab 커리큘럼 컴포넌트의 **공식 문서** 모음이다. 용어, 역할, 비유는 [`glossary.md`](glossary.md), 아키텍처 기준은 [`ref/Ref.md`](../ref/Ref.md), 복붙 명령은 [`cli/ops/`](../cli/ops/)를 참고한다.

---

## 런타임, 클러스터

| 컴포넌트 | 설명 | 공식 문서 |
| --- | --- | --- |
| k3d | Docker 기반 k3s 클러스터 도구 | [https://k3d.io/stable/](https://k3d.io/stable/) |
| Kubernetes | 컨테이너 오케스트레이션 | [https://kubernetes.io/docs/](https://kubernetes.io/docs/) |
| k3s | 경량 Kubernetes | [https://docs.k3s.io/](https://docs.k3s.io/) |
| Terraform | IaC | [https://developer.hashicorp.com/terraform/docs](https://developer.hashicorp.com/terraform/docs) |
| Helm | k8s 패키지 매니저 | [https://helm.sh/docs/](https://helm.sh/docs/) |

---

## 진입, 네트워킹

| 컴포넌트 | 설명 | 공식 문서 |
| --- | --- | --- |
| MetalLB | 베어메탈 LoadBalancer | [https://metallb.universe.tf/](https://metallb.universe.tf/) |
| ingress-nginx | Ingress 컨트롤러 | [https://kubernetes.github.io/ingress-nginx/](https://kubernetes.github.io/ingress-nginx/) |
| Kong | Ingress / API Gateway | [https://docs.konghq.com/](https://docs.konghq.com/) |
| Traefik | Ingress 컨트롤러 | [https://doc.traefik.io/traefik/](https://doc.traefik.io/traefik/) |
| HAProxy Ingress | HAProxy 기반 Ingress | [https://haproxy-ingress.github.io/](https://haproxy-ingress.github.io/) |
| Envoy Gateway | Gateway API (Envoy) | [https://gateway.envoyproxy.io/](https://gateway.envoyproxy.io/) |
| Cilium Gateway | Gateway API (Cilium) | [https://docs.cilium.io/en/stable/network/servicemesh/gateway-api/gateway-api/](https://docs.cilium.io/en/stable/network/servicemesh/gateway-api/gateway-api/) |
| Istio Gateway | Gateway API (Istio) | [https://istio.io/latest/docs/tasks/traffic-management/ingress/gateway-api/](https://istio.io/latest/docs/tasks/traffic-management/ingress/gateway-api/) |
| Gateway API | Ingress 후속 표준 | [https://gateway-api.sigs.k8s.io/](https://gateway-api.sigs.k8s.io/) |

랩 비교, 실행: [`compare/gateway-compare.md`](compare/gateway-compare.md), [`cli/ops/ingress-compare.md`](../cli/ops/ingress-compare.md)

---

## CNI, 보안, 메시

| 컴포넌트 | 설명 | 공식 문서 |
| --- | --- | --- |
| Cilium | eBPF CNI, 정책, Hubble | [https://docs.cilium.io/](https://docs.cilium.io/) |
| CiliumNetworkPolicy | L3–L7 네트워크 정책 | [https://docs.cilium.io/en/stable/security/policy/](https://docs.cilium.io/en/stable/security/policy/) |
| Istio | 서비스 메시 | [https://istio.io/latest/docs/](https://istio.io/latest/docs/) |
| PeerAuthentication | mTLS 모드 CRD | [https://istio.io/latest/docs/reference/config/security/peer_authentication/](https://istio.io/latest/docs/reference/config/security/peer_authentication/) |
| Kiali | 메시 관측 UI | [https://kiali.io/docs/](https://kiali.io/docs/) |
| cert-manager | TLS 인증서 자동화 | [https://cert-manager.io/docs/](https://cert-manager.io/docs/) |

---

## IAM, 시크릿, 레지스트리

| 컴포넌트 | 설명 | 공식 문서 |
| --- | --- | --- |
| Keycloak | OIDC/OAuth2 IdP | [https://www.keycloak.org/documentation](https://www.keycloak.org/documentation) |
| Vault | 시크릿, PKI 관리 | [https://developer.hashicorp.com/vault/docs](https://developer.hashicorp.com/vault/docs) |
| Harbor | 컨테이너 레지스트리 | [https://goharbor.io/docs/](https://goharbor.io/docs/) |

---

## GitOps, 배포

| 컴포넌트 | 설명 | 공식 문서 |
| --- | --- | --- |
| Argo CD | GitOps CD | [https://argo-cd.readthedocs.io/](https://argo-cd.readthedocs.io/) |
| Argo Rollouts | Blue/Green, Canary | [https://argo-rollouts.readthedocs.io/](https://argo-rollouts.readthedocs.io/) |
| Headlamp | k8s 웹 UI | [https://headlamp.dev/docs/latest/](https://headlamp.dev/docs/latest/) |

---

## 관측

| 컴포넌트 | 설명 | 공식 문서 |
| --- | --- | --- |
| Prometheus | 메트릭 TSDB | [https://prometheus.io/docs/](https://prometheus.io/docs/) |
| Grafana | 대시보드 UI | [https://grafana.com/docs/](https://grafana.com/docs/) |
| Loki | 로그 집계 | [https://grafana.com/docs/loki/latest/](https://grafana.com/docs/loki/latest/) |
| Fluent Bit | 로그 수집 에이전트 | [https://docs.fluentbit.io/](https://docs.fluentbit.io/) |
| Tempo | 분산 트레이스 | [https://grafana.com/docs/tempo/latest/](https://grafana.com/docs/tempo/latest/) |
| OpenTelemetry Collector | 텔레메트리 파이프 | [https://opentelemetry.io/docs/collector/](https://opentelemetry.io/docs/collector/) |
| OpenSearch | 검색, 분석 엔진 | [https://docs.opensearch.org/latest/](https://docs.opensearch.org/latest/) |

트레이스 실습: [`cli/ops/otel-trace-lab.md`](../cli/ops/otel-trace-lab.md)

---

## 데이터

| 컴포넌트 | 설명 | 공식 문서 |
| --- | --- | --- |
| MySQL | 관계형 DB | [https://dev.mysql.com/doc/](https://dev.mysql.com/doc/) |
| Redis | 인메모리 KV, 캐시 | [https://redis.io/docs/](https://redis.io/docs/) |
| Kafka | 이벤트 스트리밍 | [https://kafka.apache.org/documentation/](https://kafka.apache.org/documentation/) |
| RabbitMQ | AMQP 메시지 브로커 | [https://www.rabbitmq.com/docs](https://www.rabbitmq.com/docs) |

---

## 기타 (랩 맥락)

| 컴포넌트 | 설명 | 공식 문서 |
| --- | --- | --- |
| Gitea | Git 호스팅 (외부) | [https://docs.gitea.com/](https://docs.gitea.com/) |
| Jenkins | CI (외부) | [https://www.jenkins.io/doc/](https://www.jenkins.io/doc/) |

프록시, DNS는 공식 제품 문서 대신 랩 가이드를 쓴다: [`doc/01-proxy-and-dns.md`](01-proxy-and-dns.md), [`cli/ops/proxy.md`](../cli/ops/proxy.md)
