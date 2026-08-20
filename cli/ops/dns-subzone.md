# dns-subzone ops

`origemite.com`은 **개인용**. 이 랩 저장소·Terraform로 Route53을 건드리지 않는다. destroy/초기화는 k3d·Helm·매니페스트만 — DNS·실제 프록시 존과 분리.

나중에 랩/서비스용 도메인을 **따로 구매**하면 그때 연결한다. 자리만 [`dns/inventory.yaml`](../../dns/inventory.yaml)의 `future_lab_zone`.

지금은 이름·MetalLB 매핑 참조 + `dig` 읽기만.

## inventory

```bash
cat dns/inventory.yaml
```

## FQDN → MetalLB (클러스터)

| FQDN | MetalLB IP |
| --- | --- |
| demo.nginx.lab.origemite.com | 172.18.255.201 |
| demo.gateway.lab.origemite.com | 172.18.255.202 |
| demo.cilium.lab.origemite.com | 172.18.255.203 |
| demo.kong.lab.origemite.com | 172.18.255.204 |
| demo.traefik.lab.origemite.com | 172.18.255.205 |
| demo.istio.lab.origemite.com | 172.18.255.206 |
| demo.haproxy.lab.origemite.com | 172.18.255.207 |

프록시 Host 분기 — [`proxy.md`](proxy.md).

## 해석 확인만 (읽기)

```bash
dig +short argocd.lab.origemite.com
dig +short demo.nginx.lab.origemite.com
```

NXDOMAIN이어도 이 저장소에서 레코드를 만들지 않는다.
