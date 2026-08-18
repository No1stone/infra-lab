# infra-lab

홈랩에서 Kubernetes 운영 스택을 연습하는 저장소입니다.

공개 입구는 AWS 프록시이고, 실제 클러스터는 집 Ubuntu 노트북의 k3d에서 돌아갑니다. 프록시가 `*.lab.origemite.com`을 노트북으로 넘깁니다.

## 구조

```text
인터넷
  └─ *.lab.origemite.com
       └─ AWS 프록시 (Ubuntu, 1 vCPU / 2 GiB)
            └─ reverse SSH
                 └─ Ubuntu 노트북 (k3d)
                      ├─ Terraform
                      ├─ Helm
                      └─ Argo CD
```

## 자원

| 역할 | 머신 | 스펙 |
| --- | --- | --- |
| 보유 | MacBook Pro | macOS |
| 보유 | MacBook Air | macOS |
| 공개 입구 | AWS 프록시 | Ubuntu, 1 vCPU / 2 GiB |
| 랩 호스트 | Ubuntu 노트북 | Intel 8세대 i5, RAM 32 GiB, swap 64 GiB |

## 연결

1. Ubuntu 노트북이 AWS 프록시로 reverse SSH를 유지합니다.
2. 프록시에서 노트북으로 SSH 접속이 가능합니다.
3. 프록시가 `*.lab.origemite.com` 트래픽을 노트북으로 전달합니다.
4. 노트북의 k3d가 랩 서비스를 받아 처리합니다.

## 실습 스택

Ubuntu 노트북 k3d 위에서 다음을 연습합니다.

- Terraform
- Helm
- Argo CD

## 저장소 골격

```text
terraform/
helm/
  values/nginx.yaml        # 진입점 ingress-nginx
k8s/
  namespace/               # nginx + 워크로드 ns
  node/                    # 노드당 1 워크로드 (mysql redis kafka rabbitmq vault)
  deployment/              # replicas: 1, nodeSelector
  service/
  ingress/                 # *.lab.origemite.com
  pod/                     # 디버그용 단독 파드
  configmap/
argocd/
cli/
```

진입점은 `nginx` 네임스페이스입니다. 데이터 노드 5개는 각각 파드 1개입니다.

infra-dev 리소스 서버에서 가져온 1차 워크로드: mysql, redis, kafka, rabbitmq, vault.  
다음 후보: prometheus, grafana, elasticsearch, kibana, fluentbit, otel, loki, zipkin.

## CLI

명령은 두 계층으로 나눕니다.

- `cli/doc/` — 기본·자주 쓰는 명령. 자리표시자 `<이름>`을 씁니다.
- `cli/ops/` — 작업 중 나온 실명 명령. 복사해서 바로 실행합니다.

리소스 파일은 `k3d`, `terraform`, `helm`, `argocd`, `kube`, `proxy`입니다.
