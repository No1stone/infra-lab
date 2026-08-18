# argocd

k3d 랩의 Argo CD CLI. Ingress(`*.lab.origemite.com`) 뒤에서는 `--grpc-web`이 필요하고, 랩 인증서는 `--insecure`로 건너뛴다. in-cluster 목적지는 `https://kubernetes.default.svc`다. `-n argocd`는 kubectl/admin/`--core`용 네임스페이스이고, Application 네임스페이스는 `-N`/`--app-namespace`다.

## 공통 플래그

이후 명령에 Ingress용 gRPC-web과 인증서 검증 생략을 붙인다.

```bash
argocd <command> --server <argocd-host> --grpc-web --insecure
```

세션마다 반복하지 않으려면 환경 변수로 둔다.

```bash
export ARGOCD_OPTS='--grpc-web --insecure --server <argocd-host>'
```

토큰으로 인증한다.

```bash
export ARGOCD_AUTH_TOKEN='<token>'
```

API 서버 대신 kube API로 직접 말한다.

```bash
argocd app list --core --kube-context <k3d-context> -n argocd
```

port-forward로 CLI를 붙인다.

```bash
argocd app list --port-forward --port-forward-namespace argocd
```

TLS를 끈다.

```bash
argocd app list --plaintext --server <argocd-host>
```

## version

CLI와 서버 버전을 함께 본다.

```bash
argocd version
```

클라이언트 버전만 본다.

```bash
argocd version --client
```

짧은 형식으로 본다.

```bash
argocd version --short
```

JSON으로 본다.

```bash
argocd version -o json --grpc-web --insecure --server <argocd-host>
```

## login / logout / context

시크릿에서 초기 admin 비밀번호를 읽는다.

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d; echo
```

argocd admin으로 초기 비밀번호를 출력한다. kube 접근이 필요하다.

```bash
argocd admin initial-password -n argocd
```

Ingress 경유로 로그인한다. 랩에서 가장 흔하다.

```bash
argocd login <argocd-host> --username admin --password '<password>' --grpc-web --insecure
```

컨텍스트 이름을 지정해 로그인한다.

```bash
argocd login <argocd-host> --username admin --password '<password>' --name lab --grpc-web --insecure
```

TLS 테스트가 멈추면 건너뛴다.

```bash
argocd login <argocd-host> --username admin --password '<password>' --grpc-web --insecure --skip-test-tls
```

kube API로 직접 로그인한다. 같은 k3d면 가능하다.

```bash
argocd login --core --kube-context <k3d-context> --name lab-core
```

port-forward로 로그인한다.

```bash
argocd login localhost:8080 --username admin --password '<password>' --insecure --port-forward --port-forward-namespace argocd
```

로그아웃한다.

```bash
argocd logout <argocd-host>
```

저장된 컨텍스트 목록을 본다.

```bash
argocd context
```

컨텍스트를 전환한다.

```bash
argocd context <context-name>
```

컨텍스트를 지운다.

```bash
argocd context <context-name> --delete
```

만료된 토큰을 다시 받는다.

```bash
argocd relogin
```

비밀번호를 넣어 다시 로그인한다.

```bash
argocd relogin --password '<password>'
```

## account

계정 목록을 본다.

```bash
argocd account list
```

계정 상세를 본다.

```bash
argocd account get --account admin
```

현재 로그인 사용자 정보를 본다.

```bash
argocd account get-user-info
```

admin 비밀번호를 바꾼다.

```bash
argocd account update-password --account admin --current-password '<password>' --new-password '<new-password>'
```

계정 토큰을 만든다.

```bash
argocd account generate-token --account admin
```

만료 시간과 ID를 지정해 토큰을 만든다.

```bash
argocd account generate-token --account admin --expires-in 24h --id <token-id>
```

계정 토큰을 지운다.

```bash
argocd account delete-token <token-id> --account admin
```

현재 세션 토큰을 본다.

```bash
argocd account session-token
```

애플리케이션 조회 권한을 확인한다.

```bash
argocd account can-i get applications '*'
```

특정 앱 동기화 권한을 확인한다.

```bash
argocd account can-i sync applications <app-name>
```

저장소 생성 권한을 확인한다.

```bash
argocd account can-i create repositories '*'
```

비밀번호 bcrypt 해시를 만든다.

```bash
argocd account bcrypt --password '<password>'
```

## app 조회

앱 목록을 본다.

```bash
argocd app list
```

넓은 형식으로 앱 목록을 본다.

```bash
argocd app list -o wide
```

YAML로 앱 목록을 본다.

```bash
argocd app list -o yaml
```

프로젝트로 앱을 거른다.

```bash
argocd app list -p default
```

라벨로 앱을 거른다.

```bash
argocd app list -l <key>=<value>
```

Application 네임스페이스로 앱을 거른다.

```bash
argocd app list -N argocd
```

앱 상세를 본다.

```bash
argocd app get <app-name>
```

앱 상세를 YAML로 본다.

```bash
argocd app get <app-name> -o yaml
```

앱 상세를 JSON으로 본다.

```bash
argocd app get <app-name> -o json
```

앱 상태를 새로고침한다.

```bash
argocd app get <app-name> --refresh
```

캐시 없이 하드 리프레시한다.

```bash
argocd app get <app-name> --hard-refresh
```

파라미터 오버라이드를 같이 본다.

```bash
argocd app get <app-name> --show-params
```

진행 중인 작업을 같이 본다.

```bash
argocd app get <app-name> --show-operation
```

## app 생성

Git 디렉터리 앱을 만든다. k3d in-cluster 목적지다.

```bash
argocd app create <app-name> \
  --repo <repo-url> \
  --path <path> \
  --revision HEAD \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace <namespace> \
  --project default
```

클러스터 이름으로 목적지를 지정한다.

```bash
argocd app create <app-name> \
  --repo <repo-url> \
  --path <path> \
  --dest-name in-cluster \
  --dest-namespace <namespace>
```

하위 디렉터리까지 재귀한다.

```bash
argocd app create <app-name> \
  --repo <repo-url> \
  --path <path> \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace <namespace> \
  --directory-recurse
```

Git 경로의 Helm 앱을 만든다.

```bash
argocd app create <app-name> \
  --repo <repo-url> \
  --path <path> \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace <namespace> \
  --helm-set replicaCount=1 \
  --values values.yaml \
  --release-name <release-name>
```

Helm 차트 저장소에서 앱을 만든다.

```bash
argocd app create <app-name> \
  --repo <helm-repo-url> \
  --helm-chart <chart-name> \
  --revision <chart-version> \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace <namespace>
```

Kustomize 앱을 만든다.

```bash
argocd app create <app-name> \
  --repo <repo-url> \
  --path <path> \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace <namespace> \
  --kustomize-image <image>:<tag>
```

YAML 매니페스트로 앱을 만든다.

```bash
argocd app create --file <app.yaml>
```

같은 이름의 앱이 있으면 덮어쓴다.

```bash
argocd app create <app-name> --file <app.yaml> --upsert
```

자동 동기화로 앱을 만든다.

```bash
argocd app create <app-name> \
  --repo <repo-url> \
  --path <path> \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace <namespace> \
  --sync-policy automated \
  --auto-prune \
  --self-heal
```

## app 동기화 / 대기 / 삭제

앱을 동기화한다.

```bash
argocd app sync <app-name>
```

Git에 없는 리소스를 지운다.

```bash
argocd app sync <app-name> --prune
```

적용하지 않고 미리 본다.

```bash
argocd app sync <app-name> --dry-run
```

완료를 기다리지 않고 동기화를 시작한다.

```bash
argocd app sync <app-name> --async
```

강제 적용한다.

```bash
argocd app sync <app-name> --force
```

apply 대신 replace로 동기화한다.

```bash
argocd app sync <app-name> --replace
```

타임아웃을 두고 동기화한다.

```bash
argocd app sync <app-name> --timeout 300
```

특정 리비전으로 동기화한다.

```bash
argocd app sync <app-name> --revision <git-sha>
```

OutOfSync 리소스만 동기화한다.

```bash
argocd app sync <app-name> --apply-out-of-sync-only
```

diff를 보고 확인한 뒤 동기화한다.

```bash
argocd app sync <app-name> --preview-changes
```

특정 Service만 동기화한다.

```bash
argocd app sync <app-name> --resource :Service:<svc-name>
```

특정 Deployment만 동기화한다.

```bash
argocd app sync <app-name> --resource apps:Deployment:<deploy-name>
```

로컬 디렉터리로 동기화한다. Git 조회를 하지 않는다.

```bash
argocd app sync <app-name> --local <local-path>
```

여러 앱을 한 번에 동기화한다.

```bash
argocd app sync <app-name> <other-app>
```

라벨로 자식 앱을 동기화한다.

```bash
argocd app sync -l app.kubernetes.io/instance=<parent-app>
```

프로젝트의 앱을 동기화한다.

```bash
argocd app sync --project default
```

Synced이고 Healthy가 될 때까지 기다린다.

```bash
argocd app wait <app-name>
```

헬스와 동기화를 타임아웃과 함께 기다린다.

```bash
argocd app wait <app-name> --health --sync --timeout 300
```

진행 중인 작업이 끝날 때까지 기다린다.

```bash
argocd app wait <app-name> --operation
```

삭제가 끝날 때까지 기다린다.

```bash
argocd app wait <app-name> --delete
```

Suspended가 될 때까지 기다린다.

```bash
argocd app wait <app-name> --suspended
```

특정 리소스만 기다린다.

```bash
argocd app wait <app-name> --resource apps:Deployment:<deploy-name>
```

라벨로 여러 앱을 기다린다.

```bash
argocd app wait -l app.kubernetes.io/instance=<parent-app>
```

앱을 삭제한다.

```bash
argocd app delete <app-name>
```

확인 없이 앱을 삭제한다.

```bash
argocd app delete <app-name> --yes
```

클러스터 리소스까지 같이 지운다.

```bash
argocd app delete <app-name> --cascade
```

앱 CR만 지우고 클러스터 리소스는 남긴다.

```bash
argocd app delete <app-name> --cascade=false
```

foreground로 삭제한다.

```bash
argocd app delete <app-name> --propagation-policy foreground
```

background로 삭제한다.

```bash
argocd app delete <app-name> --propagation-policy background
```

리소스를 orphan으로 남기고 앱만 지운다.

```bash
argocd app delete <app-name> --propagation-policy orphan
```

## app set

저장소 URL을 바꾼다.

```bash
argocd app set <app-name> --repo <repo-url>
```

매니페스트 경로를 바꾼다.

```bash
argocd app set <app-name> --path <path>
```

추적 브랜치나 태그를 바꾼다.

```bash
argocd app set <app-name> --revision <branch-or-tag>
```

커밋 SHA로 고른다.

```bash
argocd app set <app-name> --revision <git-sha>
```

in-cluster 목적지로 바꾼다.

```bash
argocd app set <app-name> --dest-server https://kubernetes.default.svc
```

클러스터 이름으로 목적지를 바꾼다.

```bash
argocd app set <app-name> --dest-name in-cluster
```

배포 네임스페이스를 바꾼다.

```bash
argocd app set <app-name> --dest-namespace <namespace>
```

프로젝트를 바꾼다.

```bash
argocd app set <app-name> --project <project-name>
```

자동 동기화를 켠다.

```bash
argocd app set <app-name> --sync-policy automated
```

수동 동기화로 되돌린다.

```bash
argocd app set <app-name> --sync-policy none
```

prune과 self-heal을 켠다.

```bash
argocd app set <app-name> --auto-prune --self-heal
```

prune을 끈다.

```bash
argocd app set <app-name> --sync-option Prune=false
```

없는 네임스페이스를 만들게 한다.

```bash
argocd app set <app-name> --sync-option CreateNamespace=true
```

sync option을 제거한다.

```bash
argocd app set <app-name> --sync-option '!Prune=false'
```

Helm values 파일을 지정한다.

```bash
argocd app set <app-name> --values values.yaml
```

여러 Helm values 파일을 지정한다.

```bash
argocd app set <app-name> --values values.yaml --values values-lab.yaml
```

Helm 값을 덮어쓴다.

```bash
argocd app set <app-name> --helm-set replicaCount=2
```

Helm 문자열 값을 덮어쓴다.

```bash
argocd app set <app-name> --helm-set-string image.tag=<tag>
```

파일에서 Helm 값을 넣는다.

```bash
argocd app set <app-name> --helm-set-file <key>=<file-path>
```

Helm 릴리스 이름을 바꾼다.

```bash
argocd app set <app-name> --release-name <release-name>
```

Helm 차트 이름을 바꾼다.

```bash
argocd app set <app-name> --helm-chart <chart-name>
```

없는 values 파일을 무시한다.

```bash
argocd app set <app-name> --ignore-missing-value-files
```

파라미터 오버라이드를 넣는다.

```bash
argocd app set <app-name> -p <component>.image.tag=<tag>
```

Kustomize 이미지를 바꾼다.

```bash
argocd app set <app-name> --kustomize-image <image>:<tag>
```

Kustomize nameprefix를 넣는다.

```bash
argocd app set <app-name> --nameprefix <prefix>
```

Kustomize namesuffix를 넣는다.

```bash
argocd app set <app-name> --namesuffix <suffix>
```

Kustomize 네임스페이스를 넣는다.

```bash
argocd app set <app-name> --kustomize-namespace <namespace>
```

멀티소스의 위치 1번 저장소를 바꾼다.

```bash
argocd app set <app-name> --source-position 1 --repo <repo-url>
```

소스 이름으로 리비전을 바꾼다.

```bash
argocd app set <app-name> --source-name <source-name> --revision <revision>
```

## app unset

파라미터 오버라이드를 지운다.

```bash
argocd app unset <app-name> -p <component>.image.tag
```

Helm values 파일 지정을 지운다.

```bash
argocd app unset <app-name> --values values-lab.yaml
```

literal Helm values를 지운다.

```bash
argocd app unset <app-name> --values-literal
```

Kustomize 이미지 오버라이드를 지운다.

```bash
argocd app unset <app-name> --kustomize-image <image>
```

namesuffix를 지운다.

```bash
argocd app unset <app-name> --namesuffix
```

nameprefix를 지운다.

```bash
argocd app unset <app-name> --nameprefix
```

Kustomize 네임스페이스 오버라이드를 지운다.

```bash
argocd app unset <app-name> --kustomize-namespace
```

ignore-missing-value-files를 끈다.

```bash
argocd app unset <app-name> --ignore-missing-value-files
```

특정 소스의 values를 지운다.

```bash
argocd app unset <app-name> --source-name <source-name> --values values.yaml
```

## app history / rollback / diff / manifests / resources / logs

배포 이력을 본다.

```bash
argocd app history <app-name>
```

넓은 형식으로 이력을 본다.

```bash
argocd app history <app-name> -o wide
```

바로 이전 배포로 롤백한다.

```bash
argocd app rollback <app-name>
```

이력 ID로 롤백한다.

```bash
argocd app rollback <app-name> <history-id>
```

롤백하면서 여분 리소스를 지운다.

```bash
argocd app rollback <app-name> <history-id> --prune
```

desired와 live 차이를 본다.

```bash
argocd app diff <app-name>
```

특정 리비전과 비교한다.

```bash
argocd app diff <app-name> --revision HEAD
```

커밋 SHA와 비교한다.

```bash
argocd app diff <app-name> --revision <git-sha>
```

로컬 디렉터리와 비교한다.

```bash
argocd app diff <app-name> --local <local-path>
```

캐시 없이 diff한다.

```bash
argocd app diff <app-name> --hard-refresh
```

렌더된 매니페스트를 출력한다.

```bash
argocd app manifests <app-name>
```

Git 소스 매니페스트를 출력한다.

```bash
argocd app manifests <app-name> --source git
```

클러스터 live 매니페스트를 출력한다.

```bash
argocd app manifests <app-name> --source live
```

특정 리비전의 매니페스트를 출력한다.

```bash
argocd app manifests <app-name> --revision <git-sha>
```

앱 리소스 목록을 본다.

```bash
argocd app resources <app-name>
```

고아 리소스도 본다.

```bash
argocd app resources <app-name> --orphaned
```

특정 Deployment live 매니페스트를 본다.

```bash
argocd app get-resource <app-name> --kind Deployment --resource-name <deploy-name>
```

특정 Service의 일부 필드만 본다.

```bash
argocd app get-resource <app-name> --kind Service --resource-name <svc-name> --filter-fields metadata.name,status
```

앱의 특정 리소스를 지운다.

```bash
argocd app delete-resource <app-name> --kind Deployment --resource-name <deploy-name>
```

앱 리소스를 패치한다.

```bash
argocd app patch-resource <app-name> --kind Deployment --resource-name <deploy-name> --patch '{"spec":{"replicas":1}}' --patch-type application/merge-patch+json
```

앱 파드 로그를 본다.

```bash
argocd app logs <app-name>
```

로그를 따라간다.

```bash
argocd app logs <app-name> --follow
```

최근 줄만 본다.

```bash
argocd app logs <app-name> --tail 100
```

최근 초 단위 로그만 본다.

```bash
argocd app logs <app-name> --since-seconds 600
```

컨테이너를 지정한다.

```bash
argocd app logs <app-name> --container <container>
```

특정 Deployment 로그만 본다.

```bash
argocd app logs <app-name> --kind Deployment --name <deploy-name>
```

로그를 문자열로 거른다.

```bash
argocd app logs <app-name> --filter <text>
```

이전 컨테이너 로그를 본다.

```bash
argocd app logs <app-name> --previous
```

진행 중인 작업을 중단한다.

```bash
argocd app terminate-op <app-name>
```

에디터로 앱을 고친다.

```bash
argocd app edit <app-name>
```

앱 스펙을 패치한다.

```bash
argocd app patch <app-name> --patch '{"spec":{"syncPolicy":{"automated":{"prune":true}}}}' --type merge
```

리소스 액션 목록을 본다.

```bash
argocd app actions list <app-name>
```

Deployment를 재시작한다.

```bash
argocd app actions run <app-name> restart --kind Deployment --resource-name <deploy-name>
```

삭제/prune 확인을 승인한다.

```bash
argocd app confirm-deletion <app-name>
```

멀티소스에 소스를 추가한다.

```bash
argocd app add-source <app-name> --repo <repo-url> --path <path>
```

멀티소스에서 소스를 뺀다.

```bash
argocd app remove-source <app-name> --source-position 2
```

## appset

ApplicationSet 목록을 본다.

```bash
argocd appset list
```

YAML로 ApplicationSet 목록을 본다.

```bash
argocd appset list -o yaml
```

ApplicationSet 상세를 본다.

```bash
argocd appset get <appset-name>
```

ApplicationSet을 YAML로 본다.

```bash
argocd appset get <appset-name> -o yaml
```

YAML로 ApplicationSet을 만든다.

```bash
argocd appset create <appset.yaml>
```

같은 이름이 있으면 덮어쓴다.

```bash
argocd appset create <appset.yaml> --upsert
```

생성하지 않고 렌더된 앱만 본다.

```bash
argocd appset generate <appset.yaml>
```

ApplicationSet을 지운다.

```bash
argocd appset delete <appset-name>
```

확인 없이 ApplicationSet을 지운다.

```bash
argocd appset delete <appset-name> --yes
```

네임스페이스를 붙여 ApplicationSet을 본다.

```bash
argocd appset get argocd/<appset-name>
```

## repo

등록된 저장소 목록을 본다.

```bash
argocd repo list
```

YAML로 저장소 목록을 본다.

```bash
argocd repo list -o yaml
```

저장소 상세를 본다.

```bash
argocd repo get <repo-url>
```

공개 Git 저장소를 등록한다.

```bash
argocd repo add <repo-url>
```

사용자/비밀번호로 Git 저장소를 등록한다.

```bash
argocd repo add <repo-url> --username <username> --password '<password>'
```

SSH 키로 Git 저장소를 등록한다.

```bash
argocd repo add <repo-url> --ssh-private-key-path <key-path>
```

비표준 SSH 포트 저장소를 등록한다.

```bash
argocd repo add ssh://git@<git-host>:2222/<org>/<repo>.git --ssh-private-key-path <key-path>
```

서버 인증서 검증을 끄고 등록한다.

```bash
argocd repo add <repo-url> --insecure-skip-server-verification
```

공개 Helm 저장소를 등록한다.

```bash
argocd repo add <helm-repo-url> --type helm --name <repo-name>
```

비공개 Helm 저장소를 등록한다.

```bash
argocd repo add <helm-repo-url> --type helm --name <repo-name> --username <username> --password '<password>'
```

OCI 저장소를 등록한다.

```bash
argocd repo add oci://<oci-host> --type oci --name <repo-name> --username <username> --password '<password>'
```

프로젝트에 묶인 저장소를 등록한다.

```bash
argocd repo add <repo-url> --project <project-name>
```

같은 저장소가 있으면 덮어쓴다.

```bash
argocd repo add <repo-url> --upsert
```

저장소를 제거한다.

```bash
argocd repo rm <repo-url>
```

저장소 자격증명 템플릿 목록을 본다.

```bash
argocd repocreds list
```

같은 자격증명을 여러 repo에 쓰는 템플릿을 등록한다.

```bash
argocd repocreds add <repo-url-prefix> --username <username> --password '<password>'
```

자격증명 템플릿을 지운다.

```bash
argocd repocreds rm <repo-url-prefix>
```

## cluster

kubeconfig 컨텍스트 이름을 본다.

```bash
kubectl config get-contexts -o name
```

Argo CD에 등록된 클러스터를 본다. k3d면 in-cluster가 이미 있다.

```bash
argocd cluster list
```

넓은 형식으로 클러스터를 본다.

```bash
argocd cluster list -o wide
```

JSON으로 클러스터를 본다.

```bash
argocd cluster list -o json
```

in-cluster 상세를 본다.

```bash
argocd cluster get in-cluster
```

in-cluster 서버 URL로 본다.

```bash
argocd cluster get https://kubernetes.default.svc
```

클러스터 상세를 YAML로 본다.

```bash
argocd cluster get <cluster-name> -o yaml
```

kubeconfig 컨텍스트를 Argo CD에 등록한다. 같은 k3d면 보통 필요 없다.

```bash
argocd cluster add <k3d-context>
```

이름과 확인 생략으로 클러스터를 등록한다.

```bash
argocd cluster add <k3d-context> --name <cluster-name> --yes
```

허용 네임스페이스를 제한해 등록한다.

```bash
argocd cluster add <k3d-context> --namespace <namespace>
```

같은 클러스터가 있으면 덮어쓴다.

```bash
argocd cluster add <k3d-context> --upsert
```

클러스터 표시 이름을 바꾼다.

```bash
argocd cluster set <cluster-name> --name <new-name>
```

모든 네임스페이스를 허용한다.

```bash
argocd cluster set <cluster-name> --namespace '*'
```

클러스터 인증을 갱신한다.

```bash
argocd cluster rotate-auth <cluster-name>
```

클러스터 자격증명을 제거한다.

```bash
argocd cluster rm <cluster-name>
```

서버 URL로 클러스터를 제거한다.

```bash
argocd cluster rm https://kubernetes.default.svc
```

## proj

프로젝트 목록을 본다.

```bash
argocd proj list
```

YAML로 프로젝트 목록을 본다.

```bash
argocd proj list -o yaml
```

기본 프로젝트 상세를 본다.

```bash
argocd proj get default
```

프로젝트 상세를 본다.

```bash
argocd proj get <project-name>
```

프로젝트를 YAML로 본다.

```bash
argocd proj get <project-name> -o yaml
```

프로젝트를 만든다.

```bash
argocd proj create <project-name>
```

설명과 함께 프로젝트를 만든다.

```bash
argocd proj create <project-name> --description '<text>'
```

허용 repo와 in-cluster dest를 넣어 만든다.

```bash
argocd proj create <project-name> \
  -s <repo-url> \
  -d https://kubernetes.default.svc,<namespace>
```

YAML로 프로젝트를 만든다.

```bash
argocd proj create <project-name> -f <project.yaml>
```

허용 소스를 설정한다.

```bash
argocd proj set <project-name> -s <repo-url>
```

허용 소스를 추가한다.

```bash
argocd proj add-source <project-name> <repo-url>
```

허용 소스를 뺀다.

```bash
argocd proj remove-source <project-name> <repo-url>
```

in-cluster 목적지를 추가한다.

```bash
argocd proj add-destination <project-name> https://kubernetes.default.svc <namespace>
```

모든 네임스페이스를 목적지로 허용한다.

```bash
argocd proj add-destination <project-name> https://kubernetes.default.svc '*'
```

목적지를 뺀다.

```bash
argocd proj remove-destination <project-name> https://kubernetes.default.svc <namespace>
```

클러스터 스코프 리소스를 허용한다.

```bash
argocd proj allow-cluster-resource <project-name> '*' '*'
```

Namespace 생성을 막는다.

```bash
argocd proj deny-cluster-resource <project-name> '*' Namespace
```

네임스페이스 스코프 리소스를 허용한다.

```bash
argocd proj allow-namespace-resource <project-name> '*' '*'
```

ResourceQuota를 막는다.

```bash
argocd proj deny-namespace-resource <project-name> '*' ResourceQuota
```

에디터로 프로젝트를 고친다.

```bash
argocd proj edit <project-name>
```

sync window 목록을 본다.

```bash
argocd proj windows list <project-name>
```

프로젝트 롤 목록을 본다.

```bash
argocd proj role list <project-name>
```

프로젝트 롤을 만든다.

```bash
argocd proj role create <project-name> <role-name>
```

롤에 get 정책을 넣는다.

```bash
argocd proj role add-policy <project-name> <role-name> --action get --permission allow --object '*'
```

프로젝트를 지운다.

```bash
argocd proj delete <project-name>
```

## admin

`argocd admin`은 Kubernetes 접근이 필요하다. `-n argocd`를 붙인다.

초기 admin 비밀번호를 출력한다.

```bash
argocd admin initial-password -n argocd
```

초기 admin 비밀번호를 재생성한다.

```bash
argocd admin initial-password reset -n argocd
```

로컬에서 Argo CD UI를 연다.

```bash
argocd admin dashboard -n argocd
```

포트를 지정해 UI를 연다.

```bash
argocd admin dashboard -n argocd --port 8080
```

Argo CD 설정을 내보낸다.

```bash
argocd admin export -n argocd > argocd-backup.yaml
```

파일에서 Argo CD 설정을 가져온다.

```bash
argocd admin import -n argocd argocd-backup.yaml
```

stdin에서 Argo CD 설정을 가져온다.

```bash
argocd admin import -n argocd -
```

설정을 검증한다.

```bash
argocd admin settings validate -n argocd
```

RBAC 정책을 검증한다.

```bash
argocd admin settings rbac validate --policy-file <policy.csv> -n argocd
```

클러스터 네임스페이스 정보를 본다.

```bash
argocd admin cluster namespaces <cluster-name> -n argocd
```

Redis 초기 비밀번호가 있는지 확인한다.

```bash
argocd admin redis-initial-password -n argocd
```

argocd 파드를 본다.

```bash
kubectl -n argocd get pods
```

argocd 서비스를 본다.

```bash
kubectl -n argocd get svc
```

argocd Ingress를 본다.

```bash
kubectl -n argocd get ingress
```

초기 비밀번호 시크릿을 지운다. 비밀번호를 바꾼 뒤에 한다.

```bash
kubectl -n argocd delete secret argocd-initial-admin-secret
```

argocd-server를 로컬로 포워드한다.

```bash
kubectl -n argocd port-forward svc/argocd-server 8080:443
```
