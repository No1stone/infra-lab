# dns-subzone ops

랩 공개 호스트는 **3뎁스만**: `<앱>.<게이트웨이>.lab.origemite.com`. 
2뎁스(`rabbitmq.lab.origemite.com` 등)는 쓰지 않는다.

`origemite.com`은 개인용 — Route53 변경, destroy 영향 없음. 이름만 [`dns/inventory.yaml`](../../dns/inventory.yaml).

## inventory

```bash
cat dns/inventory.yaml
```

## 플랫폼 (nginx / .201)

| FQDN | MetalLB |
| --- | --- |
| argocd.nginx.lab.origemite.com | 172.18.255.201 |
| grafana.nginx.lab.origemite.com | 172.18.255.201 |
| rabbitmq.nginx.lab.origemite.com | 172.18.255.201 |
| vault.nginx.lab.origemite.com | 172.18.255.201 |
| … | 172.18.255.201 |

## Phase 7 demo

| FQDN | MetalLB |
| --- | --- |
| demo.nginx.lab.origemite.com | 172.18.255.201 |
| demo.gateway.lab.origemite.com | 172.18.255.202 |
| demo.cilium.lab.origemite.com | 172.18.255.203 |
| demo.kong.lab.origemite.com | 172.18.255.204 |
| demo.traefik.lab.origemite.com | 172.18.255.205 |
| demo.istio.lab.origemite.com | 172.18.255.206 |
| demo.haproxy.lab.origemite.com | 172.18.255.207 |

프록시 — [`proxy.md`](proxy.md).

## 해석 확인만 (읽기)

```bash
dig +short argocd.nginx.lab.origemite.com
dig +short rabbitmq.nginx.lab.origemite.com
dig +short demo.nginx.lab.origemite.com
```
