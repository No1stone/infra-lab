# helm

k3d에 차트를 설치할 때 쓰는 Helm CLI. 실행 위치는 Ubuntu 노트북이다.

Helm 4에서는 실패 시 롤백 플래그가 `--atomic`에서 `--rollback-on-failure`로 바뀌었다. `--atomic`은 deprecated이며 같은 동작이다.

`helm diff`는 공식 명령이 아니다. databus23/helm-diff 플러그인이다. 공식으로는 `--dry-run`을 쓴다.

## 버전 / 환경

Helm 클라이언트 버전을 짧게 출력한다.

```bash
helm version --short
```

Helm 클라이언트, Go 버전을 자세히 출력한다.

```bash
helm version
```

Helm이 쓰는 경로, 환경 변수를 출력한다.

```bash
helm env
```

도움말을 본다.

```bash
helm help
```

하위 명령 도움말을 본다.

```bash
helm install --help
```

kubeconfig 컨텍스트를 지정한다. k3d는 보통 `k3d-<클러스터>`.

```bash
helm list --kube-context k3d-<클러스터>
```

kubeconfig 파일을 지정한다.

```bash
helm list --kubeconfig <kubeconfig경로>
```

디버그 출력을 켠다.

```bash
helm list --debug
```

zsh 자동완성을 만든다.

```bash
helm completion zsh
```

지정한 k3d 컨텍스트의 모든 네임스페이스 릴리스를 본다.

```bash
helm list -A --kube-context k3d-<클러스터>
```

## 리포지토리

차트 리포를 추가한다.

```bash
helm repo add <리포> <리포URL>
```

이미 있으면 덮어쓴다.

```bash
helm repo add <리포> <리포URL> --force-update
```

등록된 리포 목록을 본다.

```bash
helm repo list
```

리포 목록을 YAML로 본다.

```bash
helm repo list -o yaml
```

모든 리포 인덱스를 갱신한다.

```bash
helm repo update
```

특정 리포만 갱신한다.

```bash
helm repo update <리포>
```

리포를 제거한다.

```bash
helm repo remove <리포>
```

패키지된 차트 디렉터리로 `index.yaml`을 만든다.

```bash
helm repo index <차트디렉터리>
```

외부에서 받을 URL을 넣어 인덱스를 만든다.

```bash
helm repo index <차트디렉터리> --url <차트배포URL>
```

ingress-nginx 공개 리포를 추가한다.

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```

cert-manager 공개 리포를 추가한다.

```bash
helm repo add jetstack https://charts.jetstack.io
```

Argo CD 공개 리포를 추가한다.

```bash
helm repo add argo https://argoproj.github.io/argo-helm
```

랩에서 쓰는 공개 리포를 한 번에 갱신한다.

```bash
helm repo update
```

## 검색

로컬에 추가한 리포에서 키워드로 찾는다.

```bash
helm search repo <키워드>
```

차트 버전을 모두 나열한다.

```bash
helm search repo <리포>/<차트> --versions
```

프리릴리스(alpha/beta/rc)도 포함한다.

```bash
helm search repo <키워드> --devel
```

버전 제약으로 찾는다.

```bash
helm search repo <리포>/<차트> --version <버전제약>
```

정규식으로 찾는다.

```bash
helm search repo '<정규식>' --regexp
```

JSON으로 찾는다.

```bash
helm search repo <키워드> -o json
```

결과가 없으면 실패로 처리한다.

```bash
helm search repo <키워드> --fail-on-no-result
```

Artifact Hub에서 찾는다. 네트워크가 필요하다.

```bash
helm search hub <키워드>
```

허브 결과와 실제 차트 리포 URL을 같이 본다. `helm repo add`할 때 쓴다.

```bash
helm search hub <키워드> --list-repo-url
```

## 차트 정보

Chart.yaml을 본다.

```bash
helm show chart <리포>/<차트>
```

기본 values.yaml을 본다. 설치 전에 거의 항상 본다.

```bash
helm show values <리포>/<차트>
```

특정 버전의 values를 본다.

```bash
helm show values <리포>/<차트> --version <버전>
```

README를 본다.

```bash
helm show readme <리포>/<차트>
```

차트에 들어 있는 CRD 매니페스트를 본다.

```bash
helm show crds <리포>/<차트>
```

차트, values, README, CRD를 모두 본다.

```bash
helm show all <리포>/<차트>
```

로컬 차트 디렉터리의 values를 본다.

```bash
helm show values <차트경로>
```

패키지 파일의 Chart.yaml을 본다.

```bash
helm show chart <차트경로>/<차트>-<버전>.tgz
```

values를 파일로 저장한다. 이후 `-f`에 쓴다.

```bash
helm show values <리포>/<차트> > values.yaml
```

## 받기 / 패키지 / 의존성

차트를 `.tgz`로 받는다.

```bash
helm pull <리포>/<차트>
```

특정 버전을 받는다.

```bash
helm pull <리포>/<차트> --version <버전>
```

받아서 바로 푼다.

```bash
helm pull <리포>/<차트> --untar
```

저장 위치와 풀 디렉터리를 지정한다.

```bash
helm pull <리포>/<차트> --destination <저장디렉터리> --untar --untardir <풀디렉터리>
```

리포를 등록하지 않고 URL로 받는다.

```bash
helm pull <차트> --repo <리포URL> --version <버전>
```

OCI 레지스트리에서 받는다.

```bash
helm pull oci://<레지스트리>/<경로>/<차트> --version <버전>
```

차트 디렉터리를 `.tgz`로 묶는다.

```bash
helm package <차트경로>
```

출력 위치를 지정한다.

```bash
helm package <차트경로> --destination <저장디렉터리>
```

패키지 전에 의존성을 갱신한다.

```bash
helm package <차트경로> --dependency-update
```

패키지 시 차트 버전을 덮어쓴다.

```bash
helm package <차트경로> --version <버전> --app-version <앱버전>
```

Chart.yaml 의존성 목록을 본다.

```bash
helm dependency list <차트경로>
```

Chart.yaml 기준으로 `charts/`를 받는다.

```bash
helm dependency update <차트경로>
```

Chart.lock 기준으로 `charts/`를 다시 만든다.

```bash
helm dependency build <차트경로>
```

## 설치 / 업그레이드 / 제거

리포 차트를 설치한다.

```bash
helm install <릴리스> <리포>/<차트> -n <네임스페이스>
```

네임스페이스가 없으면 만든다.

```bash
helm install <릴리스> <리포>/<차트> -n <네임스페이스> --create-namespace
```

values 파일로 설치한다.

```bash
helm install <릴리스> <리포>/<차트> -n <네임스페이스> --create-namespace -f values.yaml
```

values 파일을 여러 개 쓴다. 오른쪽이 이긴다.

```bash
helm install <릴리스> <리포>/<차트> -n <네임스페이스> -f values.yaml -f <오버라이드.yaml>
```

한 키만 `--set`으로 덮는다.

```bash
helm install <릴리스> <리포>/<차트> -n <네임스페이스> --set <키>=<값>
```

Ingress 호스트처럼 랩 도메인을 넣는다.

```bash
helm install <릴리스> <리포>/<차트> -n <네임스페이스> --set <ingress호스트키>=<앱>.lab.origemite.com
```

문자열로 강제한다.

```bash
helm install <릴리스> <리포>/<차트> -n <네임스페이스> --set-string <키>=<값>
```

차트 버전을 고정한다.

```bash
helm install <릴리스> <리포>/<차트> -n <네임스페이스> --version <버전>
```

리포 등록 없이 URL로 설치한다.

```bash
helm install <릴리스> <차트> --repo <리포URL> --version <버전> -n <네임스페이스>
```

로컬 차트 디렉터리로 설치한다.

```bash
helm install <릴리스> <차트경로> -n <네임스페이스>
```

로컬 `.tgz`로 설치한다.

```bash
helm install <릴리스> <차트경로>/<차트>-<버전>.tgz -n <네임스페이스>
```

릴리스 이름을 자동 생성한다.

```bash
helm install <리포>/<차트> --generate-name -n <네임스페이스>
```

준비될 때까지 기다린다. 한도는 `--timeout`.

```bash
helm install <릴리스> <리포>/<차트> -n <네임스페이스> --wait --timeout 10m
```

Job까지 기다린다.

```bash
helm install <릴리스> <리포>/<차트> -n <네임스페이스> --wait --wait-for-jobs --timeout 10m
```

실패하면 설치를 되돌린다. Helm 4 권장 플래그.

```bash
helm install <릴리스> <리포>/<차트> -n <네임스페이스> --rollback-on-failure --timeout 10m
```

Helm 3 이름. Helm 4에서는 deprecated이며 `--rollback-on-failure`와 같다.

```bash
helm install <릴리스> <리포>/<차트> -n <네임스페이스> --atomic --timeout 10m
```

클러스터에 넣지 않고 매니페스트만 본다.

```bash
helm install <릴리스> <리포>/<차트> -n <네임스페이스> --dry-run --debug
```

클라이언트만 시뮬레이션한다. 클러스터 접속이 없다.

```bash
helm install <릴리스> <리포>/<차트> -n <네임스페이스> --dry-run=client
```

서버 쪽 검증을 포함한 시뮬레이션.

```bash
helm install <릴리스> <리포>/<차트> -n <네임스페이스> --dry-run=server
```

dry-run에서 Secret 내용을 숨긴다.

```bash
helm install <릴리스> <리포>/<차트> -n <네임스페이스> --dry-run --hide-secret
```

설치 전 의존성을 갱신한다.

```bash
helm install <릴리스> <차트경로> -n <네임스페이스> --dependency-update
```

CRD 설치를 건너뛴다.

```bash
helm install <릴리스> <리포>/<차트> -n <네임스페이스> --skip-crds
```

릴리스를 업그레이드한다.

```bash
helm upgrade <릴리스> <리포>/<차트> -n <네임스페이스>
```

없으면 설치, 있으면 업그레이드. 실습에서 가장 많이 쓴다.

```bash
helm upgrade --install <릴리스> <리포>/<차트> -n <네임스페이스> --create-namespace
```

values와 wait를 붙인 업그레이드.

```bash
helm upgrade --install <릴리스> <리포>/<차트> -n <네임스페이스> --create-namespace -f values.yaml --wait --timeout 10m
```

실패 시 이전 성공 리비전으로 되돌린다.

```bash
helm upgrade --install <릴리스> <리포>/<차트> -n <네임스페이스> --rollback-on-failure --timeout 10m
```

이전 릴리스 values를 유지하고 일부만 덮는다.

```bash
helm upgrade <릴리스> <리포>/<차트> -n <네임스페이스> --reuse-values --set <키>=<값>
```

차트 기본 values로 리셋한다.

```bash
helm upgrade <릴리스> <리포>/<차트> -n <네임스페이스> --reset-values -f values.yaml
```

차트 기본값 → 직전 릴리스 값 → CLI 오버라이드 순으로 합친다.

```bash
helm upgrade <릴리스> <리포>/<차트> -n <네임스페이스> --reset-then-reuse-values --set <키>=<값>
```

업그레이드 미리보기.

```bash
helm upgrade <릴리스> <리포>/<차트> -n <네임스페이스> -f values.yaml --dry-run --debug
```

실패한 업그레이드에서 새로 생긴 리소스를 지운다.

```bash
helm upgrade <릴리스> <리포>/<차트> -n <네임스페이스> --cleanup-on-fail
```

저장할 히스토리 개수를 제한한다. 기본은 10이다.

```bash
helm upgrade <릴리스> <리포>/<차트> -n <네임스페이스> --history-max 10
```

릴리스를 제거한다.

```bash
helm uninstall <릴리스> -n <네임스페이스>
```

히스토리는 남기고 리소스만 지운다. 이후 `helm list --uninstalled`로 보인다.

```bash
helm uninstall <릴리스> -n <네임스페이스> --keep-history
```

제거 미리보기.

```bash
helm uninstall <릴리스> -n <네임스페이스> --dry-run
```

없어도 성공으로 본다.

```bash
helm uninstall <릴리스> -n <네임스페이스> --ignore-not-found
```

finalizer가 있는 리소스까지 기다린다.

```bash
helm uninstall <릴리스> -n <네임스페이스> --wait --timeout 5m --cascade foreground
```

OCI 차트를 설치한다.

```bash
helm upgrade --install <릴리스> oci://<레지스트리>/<경로>/<차트> --version <버전> -n <네임스페이스>
```

이미 있는 리소스를 Helm이 가져간다.

```bash
helm upgrade --install <릴리스> <리포>/<차트> -n <네임스페이스> --take-ownership
```

훅을 건너뛴다.

```bash
helm upgrade <릴리스> <리포>/<차트> -n <네임스페이스> --no-hooks
```

## 조회

현재 네임스페이스의 릴리스를 본다.

```bash
helm list
```

네임스페이스를 지정한다.

```bash
helm list -n <네임스페이스>
```

모든 네임스페이스를 본다. k3d에서 현황 파악용이다.

```bash
helm list -A
```

실패한 릴리스만 본다.

```bash
helm list -n <네임스페이스> --failed
```

배포된 것만 본다.

```bash
helm list -n <네임스페이스> --deployed
```

이름 필터(정규식)로 본다.

```bash
helm list -n <네임스페이스> --filter '<정규식>'
```

날짜순 정렬한다.

```bash
helm list -n <네임스페이스> --date --reverse
```

짧은 이름만 출력한다.

```bash
helm list -n <네임스페이스> -q
```

YAML로 본다.

```bash
helm list -A -o yaml
```

JSON으로 본다.

```bash
helm list -A -o json
```

`--keep-history`로 지운 릴리스를 본다.

```bash
helm list -n <네임스페이스> --uninstalled
```

릴리스 상태를 본다. 리소스, NOTES를 포함한다.

```bash
helm status <릴리스> -n <네임스페이스>
```

특정 리비전의 상태를 본다.

```bash
helm status <릴리스> -n <네임스페이스> --revision <리비전>
```

상태를 YAML로 본다.

```bash
helm status <릴리스> -n <네임스페이스> -o yaml
```

리비전 히스토리를 본다. rollback 전에 본다.

```bash
helm history <릴리스> -n <네임스페이스>
```

히스토리를 YAML로 본다.

```bash
helm history <릴리스> -n <네임스페이스> -o yaml
```

직전 리비전으로 되돌린다.

```bash
helm rollback <릴리스> -n <네임스페이스>
```

지정 리비전으로 되돌린다.

```bash
helm rollback <릴리스> <리비전> -n <네임스페이스>
```

rollback을 기다린다.

```bash
helm rollback <릴리스> <리비전> -n <네임스페이스> --wait --timeout 5m
```

rollback 미리보기.

```bash
helm rollback <릴리스> <리비전> -n <네임스페이스> --dry-run
```

릴리스에 적용된 values(사용자 오버라이드)를 본다.

```bash
helm get values <릴리스> -n <네임스페이스>
```

차트 기본값까지 합친 computed values를 본다.

```bash
helm get values <릴리스> -n <네임스페이스> --all
```

과거 리비전의 values를 본다.

```bash
helm get values <릴리스> -n <네임스페이스> --revision <리비전>
```

렌더된 매니페스트를 본다.

```bash
helm get manifest <릴리스> -n <네임스페이스>
```

NOTES를 다시 본다.

```bash
helm get notes <릴리스> -n <네임스페이스>
```

훅을 본다.

```bash
helm get hooks <릴리스> -n <네임스페이스>
```

차트 이름, 버전, 앱 버전 메타데이터를 본다.

```bash
helm get metadata <릴리스> -n <네임스페이스>
```

values, 매니페스트, 훅, NOTES를 한 번에 본다.

```bash
helm get all <릴리스> -n <네임스페이스>
```

차트에 정의된 테스트를 실행한다.

```bash
helm test <릴리스> -n <네임스페이스>
```

테스트 대기 시간을 지정한다.

```bash
helm test <릴리스> -n <네임스페이스> --timeout 5m
```

## 렌더 / 검사

클러스터에 넣지 않고 매니페스트를 렌더한다.

```bash
helm template <릴리스> <리포>/<차트>
```

values를 넣고 렌더한다.

```bash
helm template <릴리스> <리포>/<차트> -n <네임스페이스> -f values.yaml
```

`--set`과 함께 렌더한다.

```bash
helm template <릴리스> <리포>/<차트> -n <네임스페이스> --set <키>=<값>
```

로컬 차트를 렌더한다.

```bash
helm template <릴리스> <차트경로> -n <네임스페이스> -f values.yaml
```

CRD를 출력에 포함한다.

```bash
helm template <릴리스> <차트경로> --include-crds
```

특정 템플릿만 본다.

```bash
helm template <릴리스> <차트경로> -s templates/<파일>.yaml
```

파일을 디렉터리로 쓴다.

```bash
helm template <릴리스> <차트경로> --output-dir <출력디렉터리>
```

업그레이드 렌더다. `.Release.IsUpgrade=true`가 된다.

```bash
helm template <릴리스> <차트경로> --is-upgrade -f values.yaml
```

차트 문법, 관례를 검사한다.

```bash
helm lint <차트경로>
```

values를 넣고 린트한다.

```bash
helm lint <차트경로> -f values.yaml
```

경고도 실패로 본다.

```bash
helm lint <차트경로> --strict
```

서브차트까지 린트한다.

```bash
helm lint <차트경로> --with-subcharts
```

공식으로 적용 전 차이를 본다. `helm diff`가 아니다.

```bash
helm upgrade <릴리스> <리포>/<차트> -n <네임스페이스> -f values.yaml --dry-run --debug
```

## helm diff 플러그인

코어 Helm에는 `helm diff`가 없다. 업그레이드 전 unified diff는 서드파티 플러그인이다.

플러그인 목록을 본다.

```bash
helm plugin list
```

Helm 3에서 helm-diff를 설치한다.

```bash
helm plugin install https://github.com/databus23/helm-diff
```

Helm 4는 원격 플러그인 서명을 기본 검증한다. 이 플러그인은 provenance가 없어 검증을 끈다.

```bash
helm plugin install https://github.com/databus23/helm-diff --verify=false
```

업그레이드 시 바뀔 매니페스트 차이를 본다.

```bash
helm diff upgrade <릴리스> <리포>/<차트> -n <네임스페이스> -f values.yaml
```

아직 없는 릴리스도 diff한다.

```bash
helm diff upgrade <릴리스> <리포>/<차트> -n <네임스페이스> --allow-unreleased
```

두 리비전을 비교한다.

```bash
helm diff revision <릴리스> <리비전1> <리비전2> -n <네임스페이스>
```

rollback 시 차이를 본다.

```bash
helm diff rollback <릴리스> <리비전> -n <네임스페이스>
```

## 플러그인

설치된 플러그인을 본다.

```bash
helm plugin list
```

Git URL로 설치한다.

```bash
helm plugin install <플러그인URL>
```

로컬 경로로 설치한다.

```bash
helm plugin install <플러그인경로>
```

플러그인을 갱신한다.

```bash
helm plugin update <플러그인>
```

플러그인을 제거한다.

```bash
helm plugin uninstall <플러그인>
```

플러그인 서명을 검증한다.

```bash
helm plugin verify <플러그인경로>
```

플러그인 디렉터리를 아카이브로 묶는다.

```bash
helm plugin package <플러그인경로>
```

## 차트 생성

스캐폴드를 만든다. Chart.yaml, values.yaml, templates/가 생긴다.

```bash
helm create <차트이름>
```

지정 경로에 만든다.

```bash
helm create <경로>/<차트이름>
```

starter를 지정한다.

```bash
helm create <차트이름> --starter <starter이름또는경로>
```

생성 후 린트한다.

```bash
helm lint <차트이름>
```

생성한 차트를 렌더한다.

```bash
helm template <릴리스> ./<차트이름>
```

## 자주 쓰는 플래그

전역 네임스페이스. 거의 모든 릴리스 명령에 붙인다.

```bash
helm list -n <네임스페이스>
```

설치 또는 `upgrade --install` 때 네임스페이스를 만든다.

```bash
helm upgrade --install <릴리스> <리포>/<차트> -n <네임스페이스> --create-namespace
```

values 파일. 여러 번 가능하고 오른쪽이 우선이다.

```bash
helm upgrade --install <릴리스> <리포>/<차트> -n <네임스페이스> -f values.yaml
```

CLI에서 키를 덮는다. `-f`보다 우선한다.

```bash
helm upgrade --install <릴리스> <리포>/<차트> -n <네임스페이스> --set <키>=<값>
```

중첩 키를 덮는다.

```bash
helm upgrade --install <릴리스> <리포>/<차트> -n <네임스페이스> --set <상위>.<하위>=<값>
```

리스트를 덮는다.

```bash
helm upgrade --install <릴리스> <리포>/<차트> -n <네임스페이스> --set <키>={<값1>,<값2>}
```

Pod/PVC/Service가 준비될 때까지 기다린다. 기본 한도는 `5m0s`이다.

```bash
helm upgrade --install <릴리스> <리포>/<차트> -n <네임스페이스> --wait
```

개별 K8s 작업 대기 시간을 늘린다.

```bash
helm upgrade --install <릴리스> <리포>/<차트> -n <네임스페이스> --wait --timeout 10m
```

실패 시 롤백한다. Helm 4 이름이며 `--wait`가 watcher로 잡힌다.

```bash
helm upgrade --install <릴리스> <리포>/<차트> -n <네임스페이스> --rollback-on-failure --timeout 10m
```

실패 시 롤백한다. Helm 3 이름이며 deprecated다.

```bash
helm upgrade --install <릴리스> <리포>/<차트> -n <네임스페이스> --atomic --timeout 10m
```

적용하지 않고 결과만 본다.

```bash
helm upgrade --install <릴리스> <리포>/<차트> -n <네임스페이스> --dry-run
```

dry-run과 렌더 디버그를 같이 본다.

```bash
helm upgrade --install <릴리스> <리포>/<차트> -n <네임스페이스> --dry-run --debug
```

랩 실습에서 한 줄로 자주 쓰는 조합이다.

```bash
helm upgrade --install <릴리스> <리포>/<차트> \
 --version <버전> \
 -n <네임스페이스> \
 --create-namespace \
 -f values.yaml \
 --set <ingress호스트키>=<앱>.lab.origemite.com \
 --wait \
 --timeout 10m \
 --rollback-on-failure
```

적용 전 확인만 한다.

```bash
helm upgrade --install <릴리스> <리포>/<차트> \
 --version <버전> \
 -n <네임스페이스> \
 --create-namespace \
 -f values.yaml \
 --dry-run --debug
```
