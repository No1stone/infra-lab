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

MacBook Pro는 Cursor로 이 저장소를 편집하는 작업 머신입니다. 클러스터는 Mac에서 띄우지 않습니다.

## 자원

| 역할 | 머신 | 스펙 |
| --- | --- | --- |
| Cursor 작업 | MacBook Pro | macOS |
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

## CLI

명령은 두 계층으로 나눕니다.

- `cli/doc/` — 기본·자주 쓰는 명령. 자리표시자 `<이름>`을 씁니다.
- `cli/ops/` — 작업 중 나온 실명 명령. 복사해서 바로 실행합니다.

리소스 파일은 `k3d`, `terraform`, `helm`, `argocd`, `kube`, `proxy`입니다.
