# k3d

## 버전 / 도움말

k3d와 기본 k3s 버전을 본다.
```bash
k3d version
```

k3d와 기본 k3s 버전을 본다.
```bash
k3d --version
```

사용 가능한 k3s 이미지 태그를 최신순으로 나열한다.
```bash
k3d version list k3s
```

k3s 이미지 태그를 `rancher/k3s:<tag>` 형태로 본다.
```bash
k3d version list k3s --output repo --limit 10
```

k3d 릴리스 태그를 나열한다.
```bash
k3d version list k3d
```

k3d-proxy 이미지 태그를 나열한다.
```bash
k3d version list k3d-proxy
```

k3d-tools 이미지 태그를 나열한다.
```bash
k3d version list k3d-tools
```

최상위 도움말을 본다.
```bash
k3d --help
```

서브커맨드 도움말을 본다.
```bash
k3d help
```

클러스터 명령 도움말을 본다.
```bash
k3d cluster --help
```

클러스터 생성 플래그 도움말을 본다.
```bash
k3d cluster create --help
```

특정 서브커맨드 도움말을 본다.
```bash
k3d help cluster create
```

디버그 로그를 켠다.
```bash
k3d --verbose cluster list
```

트레이스 로그를 켠다.
```bash
k3d --trace cluster list
```

로그에 타임스탬프를 붙인다.
```bash
k3d --timestamps cluster list
```

bash 자동완성을 현재 세션에 로드한다.
```bash
source <(k3d completion bash)
```

## 클러스터 생성

기본 설정으로 클러스터를 만든다. 이름을 생략하면 `k3s-default`가 된다.
```bash
k3d cluster create
```

이름을 지정해 클러스터를 만든다.
```bash
k3d cluster create <cluster>
```

에이전트(워커) 수를 지정해 만든다.
```bash
k3d cluster create <cluster> --agents <n>
```

서버(컨트롤플레인) 수를 지정해 만든다. HA는 보통 3이다.
```bash
k3d cluster create <cluster> --servers <n>
```

서버 1대와 에이전트 2대로 만든다. 32GiB 랩에 흔한 구성이다.
```bash
k3d cluster create <cluster> --servers 1 --agents 2
```

k3s 이미지를 지정해 쿠버네티스 버전을 고른다.
```bash
k3d cluster create <cluster> --image <k3s-image>
```

API 포트를 호스트에 고정한다.
```bash
k3d cluster create <cluster> --api-port <host-port>
```

API를 모든 인터페이스에 연다.
```bash
k3d cluster create <cluster> --api-port 0.0.0.0:<host-port>
```

호스트 포트를 노드나 로드밸런서에 매핑한다.
```bash
k3d cluster create <cluster> --port <host-port>:<container-port>@<nodefilter>
```

HTTP 80을 로드밸런서에 연다. `*.nginx.lab.origemite.com` 등 3뎁스 Ingress 입구다.
```bash
k3d cluster create <cluster> --port 80:80@loadbalancer
```

HTTPS 443을 로드밸런서에 연다.
```bash
k3d cluster create <cluster> --port 443:443@loadbalancer
```

80과 443을 함께 연다.
```bash
k3d cluster create <cluster> --port 80:80@loadbalancer --port 443:443@loadbalancer
```

에이전트 0번에 포트를 직접 연다.
```bash
k3d cluster create <cluster> --agents 1 --port <host-port>:<container-port>@agent:0
```

NodePort 대역을 서버에 연다. Docker가 포트마다 프록시를 만들어 무거울 수 있다.
```bash
k3d cluster create <cluster> --port 30000-32767:30000-32767@server:0
```

k3s에 추가 인자를 넘긴다. 형식은 `ARG@NODEFILTER`다.
```bash
k3d cluster create <cluster> --k3s-arg "<arg>@<nodefilter>"
```

기본 Traefik Ingress를 끈다.
```bash
k3d cluster create <cluster> --k3s-arg "--disable=traefik@server:*"
```

ServiceLB(klipper)를 끈다.
```bash
k3d cluster create <cluster> --k3s-arg "--disable=servicelb@server:*"
```

API 인증서에 SAN을 넣는다. 도메인으로 API에 붙을 때 쓴다.
```bash
k3d cluster create <cluster> --k3s-arg "--tls-san=<hostname>@server:*"
```

호스트 경로를 노드에 마운트한다.
```bash
k3d cluster create <cluster> --volume <host-path>:<container-path>@<nodefilter>
```

k3s 자동 배포 매니페스트 디렉터리에 파일을 넣는다.
```bash
k3d cluster create <cluster> --volume <host-path>:/var/lib/rancher/k3s/server/manifests@server:0
```

서버가 준비될 때까지 기다린다. 기본값은 true다.
```bash
k3d cluster create <cluster> --wait
```

생성 대기 상한을 둔다. 초과하면 롤백한다.
```bash
k3d cluster create <cluster> --wait --timeout <duration>
```

기본 kubeconfig 갱신을 끈다.
```bash
k3d cluster create <cluster> --kubeconfig-update-default=false
```

현재 컨텍스트 전환을 끈다.
```bash
k3d cluster create <cluster> --kubeconfig-switch-context=false
```

설정 파일로 만든다.
```bash
k3d cluster create --config <config-file>
```

설정 파일에 CLI로 이름만 덮어쓴다.
```bash
k3d cluster create <cluster> --config <config-file>
```

클러스터 전용 레지스트리를 같이 만든다.
```bash
k3d cluster create <cluster> --registry-create <registry>
```

레지스트리 호스트 포트까지 지정해 같이 만든다.
```bash
k3d cluster create <cluster> --registry-create <registry>:0.0.0.0:<host-port>
```

이미 있는 k3d 레지스트리를 붙인다. 이름은 `k3d-` 접두사를 포함한다.
```bash
k3d cluster create <cluster> --registry-use k3d-<registry>:<host-port>
```

k3s `registries.yaml`을 넘긴다.
```bash
k3d cluster create <cluster> --registry-config <registries-file>
```

기존 Docker 네트워크에 붙인다.
```bash
k3d cluster create <cluster> --network <docker-network>
```

로드밸런서(serverlb)를 만들지 않는다.
```bash
k3d cluster create <cluster> --no-lb
```

이미지 import용 공유 볼륨을 만들지 않는다.
```bash
k3d cluster create <cluster> --no-image-volume
```

실패 시 자동 롤백을 끈다.
```bash
k3d cluster create <cluster> --no-rollback
```

서버 컨테이너 메모리 한도를 둔다.
```bash
k3d cluster create <cluster> --servers-memory <size>
```

에이전트 컨테이너 메모리 한도를 둔다.
```bash
k3d cluster create <cluster> --agents-memory <size>
```

k3s 노드 레이블을 단다.
```bash
k3d cluster create <cluster> --k3s-node-label "<key>=<value>@<nodefilter>"
```

Docker 컨테이너 레이블을 단다.
```bash
k3d cluster create <cluster> --runtime-label "<key>=<value>@<nodefilter>"
```

노드에 환경변수를 넣는다.
```bash
k3d cluster create <cluster> --env "<key>=<value>@<nodefilter>"
```

CoreDNS와 `/etc/hosts`에 호스트 별칭을 넣는다.
```bash
k3d cluster create <cluster> --host-alias <ip>:<hostname>
```

생성 과정을 자세히 본다.
```bash
k3d --verbose cluster create <cluster> --wait --timeout 120s
```

## 클러스터 시작 / 중지 / 삭제 / 목록

클러스터를 시작한다. 기본값은 `--wait true`다.
```bash
k3d cluster start <cluster>
```

시작을 기한 안에 끝낸다.
```bash
k3d cluster start <cluster> --wait --timeout <duration>
```

모든 클러스터를 시작한다.
```bash
k3d cluster start --all
```

클러스터를 중지한다. 컨테이너는 남긴다.
```bash
k3d cluster stop <cluster>
```

모든 클러스터를 중지한다.
```bash
k3d cluster stop --all
```

클러스터를 삭제한다. 노드 컨테이너와 네트워크를 제거하고 기본 kubeconfig에서도 뺀다.
```bash
k3d cluster delete <cluster>
```

모든 클러스터를 삭제한다.
```bash
k3d cluster delete --all
```

설정 파일에 적힌 클러스터를 삭제한다.
```bash
k3d cluster delete --config <config-file>
```

클러스터 목록을 본다.
```bash
k3d cluster list
```

클러스터 목록의 짧은 별칭이다.
```bash
k3d cluster ls
```

특정 클러스터만 본다. `get`은 `list`의 별칭이다.
```bash
k3d cluster get <cluster>
```

YAML로 클러스터 상세를 본다. inspect 대용이다.
```bash
k3d cluster get <cluster> --output yaml
```

JSON으로 클러스터 상세를 본다.
```bash
k3d cluster list --output json
```

테이블 헤더를 숨긴다.
```bash
k3d cluster list --no-headers
```

## kubeconfig

클러스터 kubeconfig를 표준 출력으로 본다.
```bash
k3d kubeconfig get <cluster>
```

모든 클러스터 kubeconfig를 출력한다.
```bash
k3d kubeconfig get --all
```

kubeconfig를 파일로 저장한다.
```bash
k3d kubeconfig get <cluster> > <kubeconfig-file>
```

기본 kubeconfig에 병합하고 컨텍스트를 전환한다. `write`는 `merge`의 별칭이다.
```bash
k3d kubeconfig merge <cluster> --kubeconfig-merge-default --kubeconfig-switch-context
```

지정한 파일에 병합한다.
```bash
k3d kubeconfig merge <cluster> --output <kubeconfig-file>
```

기존 파일을 내용 무시하고 덮어쓴다.
```bash
k3d kubeconfig merge <cluster> --output <kubeconfig-file> --overwrite
```

충돌 필드를 갱신한다. 기본값은 true다.
```bash
k3d kubeconfig merge <cluster> --output <kubeconfig-file> --update
```

모든 클러스터를 기본 kubeconfig에 병합한다.
```bash
k3d kubeconfig merge --all --kubeconfig-merge-default
```

클러스터별 파일을 만들고 경로를 출력한다. `KUBECONFIG=`에 쓰기 좋다.
```bash
k3d kubeconfig write <cluster>
```

방금 쓴 kubeconfig를 현재 셸에 적용한다.
```bash
export KUBECONFIG="$(k3d kubeconfig write <cluster>)"
```

컨텍스트는 바꾸지 않고 병합만 한다.
```bash
k3d kubeconfig merge <cluster> --kubeconfig-merge-default --kubeconfig-switch-context=false
```

## 노드

에이전트 노드를 기존 클러스터에 추가한다. 기본 role은 `agent`, 기본 cluster는 `k3s-default`다.
```bash
k3d node create <node> --cluster <cluster>
```

서버 노드를 추가한다.
```bash
k3d node create <node> --cluster <cluster> --role server
```

같은 스펙으로 여러 개를 만든다.
```bash
k3d node create <node> --cluster <cluster> --replicas <n>
```

노드 k3s 이미지를 지정한다.
```bash
k3d node create <node> --cluster <cluster> --image <k3s-image>
```

노드가 뜰 때까지 기다린다. 기본값은 true다.
```bash
k3d node create <node> --cluster <cluster> --wait --timeout <duration>
```

노드에 k3s 인자를 넘긴다.
```bash
k3d node create <node> --cluster <cluster> --k3s-arg "<arg>"
```

노드 메모리 한도를 둔다.
```bash
k3d node create <node> --cluster <cluster> --memory <size>
```

노드 목록을 본다.
```bash
k3d node list
```

노드 목록의 짧은 별칭이다.
```bash
k3d node ls
```

특정 노드를 본다. `k3d-` 접두사 없이도 된다.
```bash
k3d node get <node>
```

노드를 YAML로 본다. inspect 대용이다.
```bash
k3d node get <node> --output yaml
```

노드를 JSON으로 본다.
```bash
k3d node list --output json
```

헤더 없이 노드를 본다.
```bash
k3d node list --no-headers
```

중지된 노드를 시작한다.
```bash
k3d node start <node>
```

노드를 중지한다.
```bash
k3d node stop <node>
```

노드를 삭제한다.
```bash
k3d node delete <node>
```

모든 노드를 삭제한다. 클러스터까지 날아갈 수 있다.
```bash
k3d node delete --all
```

레지스트리 노드까지 같이 삭제한다.
```bash
k3d node delete --all --registries
```

로드밸런서에 포트를 나중에 추가한다. experimental이다.
```bash
k3d node edit k3d-<cluster>-serverlb --port-add <host-port>:<container-port>
```

클러스터 쪽에서 포트를 나중에 추가한다. experimental이다.
```bash
k3d cluster edit <cluster> --port-add <host-port>:<container-port>@loadbalancer
```

## 이미지 import

로컬 Docker 이미지를 클러스터에 넣는다. 기본 대상은 `k3s-default`다.
```bash
k3d image import <image>
```

대상 클러스터를 지정한다.
```bash
k3d image import <image> --cluster <cluster>
```

여러 이미지를 한 번에 넣는다.
```bash
k3d image import <image> <image2> --cluster <cluster>
```

tar 아카이브를 넣는다. 같은 이름 파일이 있으면 ARCHIVE가 우선이다.
```bash
k3d image import <archive> --cluster <cluster>
```

여러 클러스터에 넣는다.
```bash
k3d image import <image> --cluster <cluster> --cluster <cluster2>
```

import 방식을 고른다. `auto`, `direct`, `tools` 중 하나다.
```bash
k3d image import <image> --cluster <cluster> --mode <mode>
```

tools 컨테이너를 남긴다.
```bash
k3d image import <image> --cluster <cluster> --keep-tools
```

공유 볼륨의 tarball을 남긴다.
```bash
k3d image import <image> --cluster <cluster> --keep-tarball
```

이미지 명령 도움말을 본다.
```bash
k3d image --help
```

import 플래그 도움말을 본다.
```bash
k3d image import --help
```

## 레지스트리

독립 레지스트리를 만든다. 실제 컨테이너 이름은 `k3d-<registry>`다.
```bash
k3d registry create <registry>
```

호스트 포트를 지정해 만든다.
```bash
k3d registry create <registry> --port <host-port>
```

모든 인터페이스에 레지스트리 포트를 연다.
```bash
k3d registry create <registry> --port 0.0.0.0:<host-port>
```

레지스트리 이미지를 지정한다. 기본값은 `docker.io/library/registry:2`다.
```bash
k3d registry create <registry> --image <registry-image>
```

레지스트리 데이터를 호스트에 유지한다.
```bash
k3d registry create <registry> --port <host-port> --volume <host-path>:/var/lib/registry
```

Docker Hub pull-through 캐시를 만든다.
```bash
k3d registry create <registry> --port <host-port> --proxy-remote-url https://registry-1.docker.io --volume <host-path>:/var/lib/registry
```

생성 후 How-To 문구를 숨긴다.
```bash
k3d registry create <registry> --port <host-port> --no-help
```

레지스트리를 기존 Docker 네트워크에 붙인다.
```bash
k3d registry create <registry> --default-network <docker-network>
```

레지스트리 목록을 본다.
```bash
k3d registry list
```

레지스트리를 YAML로 본다.
```bash
k3d registry list --output yaml
```

특정 레지스트리를 본다.
```bash
k3d registry list <registry>
```

헤더 없이 본다.
```bash
k3d registry list --no-headers
```

레지스트리를 삭제한다.
```bash
k3d registry delete <registry>
```

모든 레지스트리를 삭제한다.
```bash
k3d registry delete --all
```

## inspect / get

k3d에는 `inspect` 서브커맨드가 없다. `get`은 `list`의 별칭이다.

클러스터를 이름으로 조회한다.
```bash
k3d cluster get <cluster>
```

클러스터를 YAML로 본다.
```bash
k3d cluster get <cluster> -o yaml
```

클러스터를 JSON으로 본다.
```bash
k3d cluster get <cluster> -o json
```

노드를 이름으로 조회한다.
```bash
k3d node get <cluster>-server-0
```

노드를 YAML로 본다.
```bash
k3d node get <cluster>-server-0 -o yaml
```

로드밸런서 노드를 본다.
```bash
k3d node get <cluster>-serverlb
```

레지스트리를 JSON으로 본다.
```bash
k3d registry list -o json
```

k3d가 만든 컨테이너를 런타임에서 본다.
```bash
docker ps --filter "label=k3d.cluster=<cluster>"
```

로드밸런서 컨테이너를 inspect한다.
```bash
docker inspect k3d-<cluster>-serverlb
```

로드밸런서 IP만 뽑는다.
```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' k3d-<cluster>-serverlb
```

서버 노드 컨테이너를 inspect한다.
```bash
docker inspect k3d-<cluster>-server-0
```

호스트에 매핑된 80/443을 확인한다.
```bash
docker port k3d-<cluster>-serverlb
```

클러스터 Docker 네트워크를 본다.
```bash
docker network inspect k3d-<cluster>
```

k3d 관련 볼륨을 본다.
```bash
docker volume ls --filter "label=app=k3d"
```

## 설정 파일

기본 k3d 설정 템플릿을 `k3d-default.yaml`로 만든다.
```bash
k3d config init
```

출력 경로를 지정한다.
```bash
k3d config init --output <config-file>
```

기존 파일을 덮어쓴다.
```bash
k3d config init --force --output <config-file>
```

설정 파일로 클러스터를 만든다.
```bash
k3d cluster create --config <config-file>
```

설정 도움말을 본다.
```bash
k3d config --help
```

## 랩 조합: 3뎁스 `*.<게이트웨이>.lab.origemite.com`

프록시가 `*.nginx.lab.origemite.com` 등 게이트웨이 서브존을 Ubuntu 노트북으로 넘긴다. 앱 호스트는 2뎁스(`*.lab.origemite.com`)를 쓰지 않는다. 노트북 호스트 80/443이 k3d 로드밸런서를 거쳐 Ingress로 간다.

실습용 클러스터를 서버 1, 에이전트 2, HTTP/HTTPS 공개로 만들고 생성될 때까지 기다린다.
```bash
k3d cluster create <cluster> \
  --servers 1 \
  --agents 2 \
  --api-port 0.0.0.0:<api-port> \
  --port 80:80@loadbalancer \
  --port 443:443@loadbalancer \
  --wait \
  --timeout 120s
```

위와 같되 Traefik을 끄고 다른 Ingress를 쓸 때 만든다.
```bash
k3d cluster create <cluster> \
  --servers 1 \
  --agents 2 \
  --port 80:80@loadbalancer \
  --port 443:443@loadbalancer \
  --k3s-arg "--disable=traefik@server:*" \
  --wait \
  --timeout 120s
```

도메인 SAN을 API 인증서에 넣고 80/443을 연다. Ingress가 아니라 kube-apiserver용이다. 앱 Ingress 호스트는 별도로 3뎁스 FQDN을 쓴다.
```bash
k3d cluster create <cluster> \
  --api-port 0.0.0.0:<api-port> \
  --port 80:80@loadbalancer \
  --port 443:443@loadbalancer \
  --k3s-arg "--tls-san=*.nginx.lab.origemite.com@server:*"
```

로컬 레지스트리를 먼저 만들고 Helm/Argo CD 이미지 연습용 클러스터에 붙인다.
```bash
k3d registry create <registry> --port 0.0.0.0:<registry-port>
```

레지스트리를 붙인 채 80/443을 연 클러스터를 만든다.
```bash
k3d cluster create <cluster> \
  --servers 1 \
  --agents 2 \
  --port 80:80@loadbalancer \
  --port 443:443@loadbalancer \
  --registry-use k3d-<registry>:<registry-port> \
  --wait
```

클러스터 생성과 동시에 레지스트리를 만든다.
```bash
k3d cluster create <cluster> \
  --port 80:80@loadbalancer \
  --port 443:443@loadbalancer \
  --registry-create <registry>:0.0.0.0:<registry-port>
```

이미 만든 클러스터의 로드밸런서에 HTTP 80을 나중에 추가한다.
```bash
k3d node edit k3d-<cluster>-serverlb --port-add 80:80
```

이미 만든 클러스터의 로드밸런서에 HTTPS 443을 나중에 추가한다.
```bash
k3d node edit k3d-<cluster>-serverlb --port-add 443:443
```

kubeconfig를 기본 파일에 넣고 컨텍스트를 맞춘다.
```bash
k3d kubeconfig merge <cluster> --kubeconfig-merge-default --kubeconfig-switch-context
```

매핑이 붙었는지 클러스터로 확인한다.
```bash
k3d cluster get <cluster>
```

매핑이 붙었는지 노드로 확인한다.
```bash
k3d node list
```

로드밸런서 호스트 포트 매핑을 확인한다.
```bash
docker port k3d-<cluster>-serverlb
```

노트북에서 HTTP Ingress 입구가 살아 있는지 본다. 프록시 앞단 확인 전에 쓴다.
```bash
curl -I http://127.0.0.1
```

노트북에서 HTTPS Ingress 입구가 살아 있는지 본다.
```bash
curl -Ik https://127.0.0.1
```

클러스터만 멈추고 컨테이너는 남긴다. 랩 재부팅 전에 쓴다.
```bash
k3d cluster stop <cluster>
```

클러스터를 다시 올린다.
```bash
k3d cluster start <cluster> --wait --timeout 120s
```

실습 클러스터를 지운다.
```bash
k3d cluster delete <cluster>
```

실습 클러스터를 80/443 로드밸런서로 처음부터 다시 만든다.
```bash
k3d cluster create <cluster> \
  --servers 1 \
  --agents 2 \
  --port 80:80@loadbalancer \
  --port 443:443@loadbalancer \
  --wait
```

메모리 한도를 둔 가벼운 구성으로 만든다. 32GiB 호스트에서 여유를 남길 때 쓴다.
```bash
k3d cluster create <cluster> \
  --servers 1 \
  --agents 1 \
  --servers-memory 4g \
  --agents-memory 4g \
  --port 80:80@loadbalancer \
  --port 443:443@loadbalancer
```

로컬에서 빌드한 앱 이미지를 클러스터에 넣는다. Ingress로 노출하기 전 단계다.
```bash
k3d image import <image> --cluster <cluster>
```

`--agents`로 워커 수를 지정한다. 기본값은 0이다.
```bash
k3d cluster create <cluster> --agents <n>
```

`--servers`로 컨트롤플레인 수를 지정한다. 기본값은 1이다.
```bash
k3d cluster create <cluster> --servers <n>
```

`--port`로 호스트 포트를 연다. 랩 공개는 `@loadbalancer`를 쓴다.
```bash
k3d cluster create <cluster> --port 80:80@loadbalancer --port 443:443@loadbalancer
```

`--api-port`로 kube-apiserver 호스트 포트를 고정한다.
```bash
k3d cluster create <cluster> --api-port <host-port>
```

`--k3s-arg`로 k3s 인자를 넘긴다. 반드시 `@<nodefilter>`를 붙인다.
```bash
k3d cluster create <cluster> --k3s-arg "<arg>@<nodefilter>"
```

`--volume`으로 바인드 마운트한다.
```bash
k3d cluster create <cluster> --volume <host-path>:<container-path>@<nodefilter>
```

`--wait`로 서버 준비를 기다린다. create와 start의 기본값은 true다.
```bash
k3d cluster create <cluster> --wait
```

`--timeout`으로 `--wait` 상한을 둔다.
```bash
k3d cluster create <cluster> --wait --timeout 120s
```

`--image`로 k3s 버전을 고른다.
```bash
k3d cluster create <cluster> --image <k3s-image>
```

`--registry-create`로 클러스터와 레지스트리를 같이 만든다.
```bash
k3d cluster create <cluster> --registry-create <registry>
```

`--registry-use`로 이미 있는 k3d 레지스트리를 붙인다.
```bash
k3d cluster create <cluster> --registry-use k3d-<registry>:<host-port>
```

`--config`로 YAML 설정 파일을 쓴다.
```bash
k3d cluster create --config <config-file>
```
