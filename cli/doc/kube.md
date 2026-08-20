# kube

k3d 실습용 기본 kubectl 명령. Secret은 이름만 조회한다.

## 클러스터 정보

클라이언트, 서버 버전 확인

```bash
kubectl version
```

클라이언트만 확인 (클러스터 접속 불필요)

```bash
kubectl version --client
```

버전을 YAML로 보기

```bash
kubectl version -o yaml
```

컨트롤 플레인, 클러스터 서비스 주소

```bash
kubectl cluster-info
```

클러스터 디버그 덤프 (용량 큼)

```bash
kubectl cluster-info dump
```

서버가 지원하는 API 리소스

```bash
kubectl api-resources
```

API 리소스 상세 (동사 포함)

```bash
kubectl api-resources -o wide
```

네임스페이스 리소스만

```bash
kubectl api-resources --namespaced=true
```

클러스터 범위 리소스만

```bash
kubectl api-resources --namespaced=false
```

이름순 정렬

```bash
kubectl api-resources --sort-by=name
```

특정 API 그룹만

```bash
kubectl api-resources --api-group=apps
```

지원 API 버전 목록

```bash
kubectl api-versions
```

리소스 필드 설명

```bash
kubectl explain pods
```

디플로이먼트 필드 설명

```bash
kubectl explain deployments
```

중첩 필드까지 재귀 설명

```bash
kubectl explain pods --recursive
```

특정 필드만 설명

```bash
kubectl explain pods.spec.containers
```

특정 API 버전으로 설명

```bash
kubectl explain deployments --api-version=apps/v1
```

Ingress 필드 설명

```bash
kubectl explain ingress.spec.rules
```

CRD 목록 (Argo CD 등 확장 API 확인)

```bash
kubectl get crd
```

## kubeconfig / 컨텍스트

컨텍스트 목록

```bash
kubectl config get-contexts
```

컨텍스트 이름만

```bash
kubectl config get-contexts -o name
```

현재 컨텍스트

```bash
kubectl config current-context
```

컨텍스트 전환 (k3d는 보통 `k3d-<클러스터명>`)

```bash
kubectl config use-context <컨텍스트명>
```

병합된 kubeconfig 보기 (인증서 원문은 출력하지 않음)

```bash
kubectl config view
```

현재 컨텍스트만 축소해서 보기

```bash
kubectl config view --minify
```

현재 컨텍스트의 클러스터 서버 URL만

```bash
kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}'
```

현재 컨텍스트의 기본 네임스페이스를 영구 설정

```bash
kubectl config set-context --current --namespace=<네임스페이스>
```

지정 컨텍스트의 네임스페이스 설정

```bash
kubectl config set-context <컨텍스트명> --namespace=<네임스페이스>
```

kubeconfig에 등록된 클러스터 이름

```bash
kubectl config get-clusters
```

kubeconfig 사용자 이름만 (자격 증명은 출력하지 않음)

```bash
kubectl config get-users
```

이번 명령만 다른 컨텍스트 사용

```bash
kubectl --context=<컨텍스트명> get nodes
```

이번 명령만 다른 kubeconfig 파일 사용

```bash
kubectl --kubeconfig=<kubeconfig경로> get nodes
```

## 네임스페이스

네임스페이스 목록

```bash
kubectl get ns
```

네임스페이스 YAML

```bash
kubectl get ns <네임스페이스> -o yaml
```

네임스페이스 생성

```bash
kubectl create namespace <네임스페이스>
```

네임스페이스 상세

```bash
kubectl describe ns <네임스페이스>
```

네임스페이스 삭제 (안의 리소스도 삭제됨)

```bash
kubectl delete ns <네임스페이스>
```

현재 컨텍스트 네임스페이스로 조회 (`-n` 생략 시)

```bash
kubectl get pods
```

특정 네임스페이스로 조회

```bash
kubectl get pods -n <네임스페이스>
```

모든 네임스페이스

```bash
kubectl get pods -A
```

## get

파드 목록

```bash
kubectl get pods
```

파드 목록 (네임스페이스 지정)

```bash
kubectl get pods -n <네임스페이스>
```

파드 상세 컬럼 (노드, IP)

```bash
kubectl get pods -o wide
```

파드 YAML

```bash
kubectl get pod <파드명> -n <네임스페이스> -o yaml
```

파드 이름만

```bash
kubectl get pods -n <네임스페이스> -o name
```

파드 감시

```bash
kubectl get pods -n <네임스페이스> -w
```

레이블로 파드 필터

```bash
kubectl get pods -n <네임스페이스> -l <키>=<값>
```

실행 중 파드만

```bash
kubectl get pods -n <네임스페이스> --field-selector=status.phase=Running
```

서비스 목록

```bash
kubectl get svc -n <네임스페이스>
```

서비스 상세 컬럼

```bash
kubectl get svc -n <네임스페이스> -o wide
```

서비스 YAML

```bash
kubectl get svc <서비스명> -n <네임스페이스> -o yaml
```

디플로이먼트 목록

```bash
kubectl get deploy -n <네임스페이스>
```

디플로이먼트 YAML

```bash
kubectl get deploy <디플로이명> -n <네임스페이스> -o yaml
```

레플리카셋

```bash
kubectl get rs -n <네임스페이스>
```

스테이트풀셋

```bash
kubectl get sts -n <네임스페이스>
```

데몬셋

```bash
kubectl get ds -n <네임스페이스>
```

잡 / 크론잡

```bash
kubectl get job,cronjob -n <네임스페이스>
```

Ingress

```bash
kubectl get ingress -n <네임스페이스>
```

Ingress 전체 네임스페이스

```bash
kubectl get ingress -A
```

IngressClass (k3d Traefik 등)

```bash
kubectl get ingressclass
```

노드

```bash
kubectl get nodes
```

노드 상세 컬럼

```bash
kubectl get nodes -o wide
```

노드 YAML

```bash
kubectl get node <노드명> -o yaml
```

이벤트 (네임스페이스)

```bash
kubectl get events -n <네임스페이스>
```

이벤트를 시간순으로

```bash
kubectl get events -n <네임스페이스> --sort-by='.lastTimestamp'
```

경고 이벤트만

```bash
kubectl get events -n <네임스페이스> --field-selector type=Warning
```

모든 네임스페이스 이벤트

```bash
kubectl get events -A
```

최근 이벤트

```bash
kubectl events -n <네임스페이스>
```

특정 파드 이벤트를 감시

```bash
kubectl events --for pod/<파드명> -n <네임스페이스> --watch
```

네임스페이스의 주요 리소스 한 번에

```bash
kubectl get all -n <네임스페이스>
```

모든 네임스페이스의 주요 리소스

```bash
kubectl get all -A
```

ConfigMap 목록

```bash
kubectl get cm -n <네임스페이스>
```

ConfigMap YAML

```bash
kubectl get cm <컨피그맵명> -n <네임스페이스> -o yaml
```

Secret 이름만 (값/`-o yaml` 사용하지 않음)

```bash
kubectl get secrets -n <네임스페이스>
```

Secret 이름만 출력

```bash
kubectl get secrets -n <네임스페이스> -o name
```

엔드포인트

```bash
kubectl get ep -n <네임스페이스>
```

서비스어카운트

```bash
kubectl get sa -n <네임스페이스>
```

PVC

```bash
kubectl get pvc -n <네임스페이스>
```

PV

```bash
kubectl get pv
```

StorageClass

```bash
kubectl get sc
```

HPA

```bash
kubectl get hpa -n <네임스페이스>
```

NetworkPolicy

```bash
kubectl get netpol -n <네임스페이스>
```

여러 종류를 한 줄에

```bash
kubectl get deploy,svc,ingress -n <네임스페이스>
```

매니페스트 파일로 조회

```bash
kubectl get -f <파일>
```

Kustomize 디렉터리로 조회

```bash
kubectl get -k <디렉터리>
```

커스텀 컬럼 (파드 이미지)

```bash
kubectl get pod <파드명> -n <네임스페이스> -o custom-columns=NAME:.metadata.name,IMAGE:.spec.containers[0].image
```

## describe

파드 상세 (이벤트 포함)

```bash
kubectl describe pod <파드명> -n <네임스페이스>
```

디플로이먼트 상세

```bash
kubectl describe deploy <디플로이명> -n <네임스페이스>
```

서비스 상세

```bash
kubectl describe svc <서비스명> -n <네임스페이스>
```

Ingress 상세

```bash
kubectl describe ingress <인그레스명> -n <네임스페이스>
```

노드 상세

```bash
kubectl describe node <노드명>
```

네임스페이스 전체 파드 상세

```bash
kubectl describe pods -n <네임스페이스>
```

레이블로 골라 상세

```bash
kubectl describe pods -n <네임스페이스> -l <키>=<값>
```

파일 기준 상세

```bash
kubectl describe -f <파일>
```

## logs

파드 로그 스냅샷

```bash
kubectl logs <파드명> -n <네임스페이스>
```

로그 스트리밍

```bash
kubectl logs -f <파드명> -n <네임스페이스>
```

이전 컨테이너 로그 (재시작 후)

```bash
kubectl logs --previous <파드명> -n <네임스페이스>
```

특정 컨테이너

```bash
kubectl logs <파드명> -c <컨테이너명> -n <네임스페이스>
```

멀티 컨테이너 전부

```bash
kubectl logs <파드명> -n <네임스페이스> --all-containers=true
```

최근 N줄

```bash
kubectl logs <파드명> -n <네임스페이스> --tail=100
```

최근 기간만

```bash
kubectl logs <파드명> -n <네임스페이스> --since=10m
```

타임스탬프 포함

```bash
kubectl logs <파드명> -n <네임스페이스> --timestamps
```

디플로이먼트 로그

```bash
kubectl logs deploy/<디플로이명> -n <네임스페이스>
```

디플로이먼트 전체 파드 로그

```bash
kubectl logs deploy/<디플로이명> -n <네임스페이스> --all-pods=true
```

레이블로 로그

```bash
kubectl logs -n <네임스페이스> -l <키>=<값> --all-containers=true
```

스트리밍 + 이전 인스턴스

```bash
kubectl logs -f --previous <파드명> -c <컨테이너명> -n <네임스페이스>
```

## exec / port-forward

파드에서 명령 실행

```bash
kubectl exec <파드명> -n <네임스페이스> -- date
```

대화형 셸

```bash
kubectl exec -it <파드명> -n <네임스페이스> -- /bin/sh
```

bash가 있는 이미지용

```bash
kubectl exec -it <파드명> -n <네임스페이스> -- /bin/bash
```

특정 컨테이너에서 실행

```bash
kubectl exec -it <파드명> -c <컨테이너명> -n <네임스페이스> -- /bin/sh
```

디플로이먼트의 첫 파드에서 실행

```bash
kubectl exec -it deploy/<디플로이명> -n <네임스페이스> -- /bin/sh
```

파드로 포트포워드

```bash
kubectl port-forward pod/<파드명> <로컬포트>:<원격포트> -n <네임스페이스>
```

디플로이먼트로 포트포워드

```bash
kubectl port-forward deploy/<디플로이명> <로컬포트>:<원격포트> -n <네임스페이스>
```

서비스로 포트포워드

```bash
kubectl port-forward svc/<서비스명> <로컬포트>:<원격포트> -n <네임스페이스>
```

로컬 임의 포트로 포워드

```bash
kubectl port-forward pod/<파드명> :<원격포트> -n <네임스페이스>
```

모든 인터페이스에 바인딩 (랩에서 다른 머신 접근 시)

```bash
kubectl port-forward --address 0.0.0.0 svc/<서비스명> <로컬포트>:<원격포트> -n <네임스페이스>
```

## apply / delete / create / replace / diff

매니페스트 적용

```bash
kubectl apply -f <파일>
```

디렉터리 전체 적용

```bash
kubectl apply -f <디렉터리>
```

재귀 적용

```bash
kubectl apply -R -f <디렉터리>
```

Kustomize 적용

```bash
kubectl apply -k <디렉터리>
```

표준입력으로 적용

```bash
cat <파일> | kubectl apply -f -
```

적용 전 클라이언트 드라이런

```bash
kubectl apply -f <파일> --dry-run=client
```

적용 전 서버 드라이런

```bash
kubectl apply -f <파일> --dry-run=server
```

드라이런 결과를 YAML로

```bash
kubectl apply -f <파일> --dry-run=client -o yaml
```

서버사이드 apply

```bash
kubectl apply --server-side -f <파일>
```

파일로 삭제

```bash
kubectl delete -f <파일>
```

Kustomize로 삭제

```bash
kubectl delete -k <디렉터리>
```

리소스 종류, 이름으로 삭제

```bash
kubectl delete pod <파드명> -n <네임스페이스>
```

디플로이먼트 삭제

```bash
kubectl delete deploy <디플로이명> -n <네임스페이스>
```

레이블로 삭제

```bash
kubectl delete pods -n <네임스페이스> -l <키>=<값>
```

해당 종류 전부 삭제

```bash
kubectl delete pods --all -n <네임스페이스>
```

빠른 삭제 (`--now` = grace 1초)

```bash
kubectl delete pod <파드명> -n <네임스페이스> --now
```

강제 삭제 (노드 장애 실습 시)

```bash
kubectl delete pod <파드명> -n <네임스페이스> --force --grace-period=0
```

그레이스 기간 지정 삭제

```bash
kubectl delete pod <파드명> -n <네임스페이스> --grace-period=30
```

캐스케이드 orphan (종속 파드 남김)

```bash
kubectl delete deploy <디플로이명> -n <네임스페이스> --cascade=orphan
```

파일로 생성 (이미 있으면 실패)

```bash
kubectl create -f <파일>
```

디플로이먼트 빠르게 생성

```bash
kubectl create deployment <디플로이명> --image=<이미지> -n <네임스페이스>
```

레플리카 지정 생성

```bash
kubectl create deployment <디플로이명> --image=<이미지> --replicas=3 -n <네임스페이스>
```

서비스 ClusterIP 생성

```bash
kubectl create service clusterip <서비스명> --tcp=<포트>:<포트> -n <네임스페이스>
```

ConfigMap 리터럴 생성

```bash
kubectl create configmap <컨피그맵명> --from-literal=<키>=<값> -n <네임스페이스>
```

파일로 ConfigMap 생성

```bash
kubectl create configmap <컨피그맵명> --from-file=<파일> -n <네임스페이스>
```

완전 교체

```bash
kubectl replace -f <파일>
```

강제 교체 (삭제 후 재생성)

```bash
kubectl replace --force -f <파일>
```

적용 전 live vs 파일 diff

```bash
kubectl diff -f <파일>
```

Kustomize diff

```bash
kubectl diff -k <디렉터리>
```

## rollout

롤아웃 상태 감시

```bash
kubectl rollout status deploy/<디플로이명> -n <네임스페이스>
```

감시 없이 한 번만 상태 확인

```bash
kubectl rollout status deploy/<디플로이명> -n <네임스페이스> --watch=false
```

타임아웃 있는 상태 확인

```bash
kubectl rollout status deploy/<디플로이명> -n <네임스페이스> --timeout=120s
```

재시작 (새 ReplicaSet)

```bash
kubectl rollout restart deploy/<디플로이명> -n <네임스페이스>
```

레이블로 여러 디플로이먼트 재시작

```bash
kubectl rollout restart deploy -n <네임스페이스> -l <키>=<값>
```

이전 리비전으로 되돌리기

```bash
kubectl rollout undo deploy/<디플로이명> -n <네임스페이스>
```

특정 리비전으로 되돌리기

```bash
kubectl rollout undo deploy/<디플로이명> -n <네임스페이스> --to-revision=<리비전>
```

롤아웃 히스토리

```bash
kubectl rollout history deploy/<디플로이명> -n <네임스페이스>
```

특정 리비전 상세

```bash
kubectl rollout history deploy/<디플로이명> -n <네임스페이스> --revision=<리비전>
```

롤아웃 일시정지

```bash
kubectl rollout pause deploy/<디플로이명> -n <네임스페이스>
```

롤아웃 재개

```bash
kubectl rollout resume deploy/<디플로이명> -n <네임스페이스>
```

DaemonSet 롤아웃 상태

```bash
kubectl rollout status ds/<데몬셋명> -n <네임스페이스>
```

StatefulSet 롤아웃 상태

```bash
kubectl rollout status sts/<스테이트풀셋명> -n <네임스페이스>
```

## scale / autoscale

레플리카 변경

```bash
kubectl scale deploy/<디플로이명> --replicas=<개수> -n <네임스페이스>
```

현재 레플리카가 N일 때만 스케일

```bash
kubectl scale deploy/<디플로이명> --current-replicas=<현재개수> --replicas=<개수> -n <네임스페이스>
```

StatefulSet 스케일

```bash
kubectl scale sts/<스테이트풀셋명> --replicas=<개수> -n <네임스페이스>
```

파일 대상 스케일

```bash
kubectl scale --replicas=<개수> -f <파일>
```

HPA 생성 (CPU)

```bash
kubectl autoscale deploy <디플로이명> --min=<최소> --max=<최대> --cpu=<퍼센트>% -n <네임스페이스>
```

HPA 생성 (기본 정책)

```bash
kubectl autoscale deploy <디플로이명> --min=1 --max=5 -n <네임스페이스>
```

HPA 확인

```bash
kubectl get hpa <디플로이명> -n <네임스페이스>
```

HPA 삭제

```bash
kubectl delete hpa <디플로이명> -n <네임스페이스>
```

## top

파드 CPU/메모리 (metrics-server 필요)

```bash
kubectl top pod -n <네임스페이스>
```

모든 네임스페이스 파드 사용량

```bash
kubectl top pod -A
```

컨테이너별 사용량

```bash
kubectl top pod <파드명> -n <네임스페이스> --containers
```

레이블로 사용량

```bash
kubectl top pod -n <네임스페이스> -l <키>=<값>
```

노드 사용량

```bash
kubectl top node
```

특정 노드 사용량

```bash
kubectl top node <노드명>
```

## drain / cordon / uncordon

스케줄 중지 (파드는 유지)

```bash
kubectl cordon <노드명>
```

스케줄 재개

```bash
kubectl uncordon <노드명>
```

노드 드레인 (k3d 워커 점검 실습)

```bash
kubectl drain <노드명> --ignore-daemonsets --delete-emptydir-data
```

비관리 파드가 있어도 드레인

```bash
kubectl drain <노드명> --ignore-daemonsets --delete-emptydir-data --force
```

그레이스 기간 지정 드레인

```bash
kubectl drain <노드명> --ignore-daemonsets --delete-emptydir-data --grace-period=60
```

드레인 드라이런

```bash
kubectl drain <노드명> --ignore-daemonsets --delete-emptydir-data --dry-run=client
```

노드 스케줄 가능 여부 확인

```bash
kubectl get nodes
```

## wait

Ready 될 때까지

```bash
kubectl wait --for=condition=Ready pod/<파드명> -n <네임스페이스> --timeout=60s
```

디플로이먼트 Available

```bash
kubectl wait --for=condition=Available deploy/<디플로이명> -n <네임스페이스> --timeout=120s
```

삭제될 때까지

```bash
kubectl wait --for=delete pod/<파드명> -n <네임스페이스> --timeout=60s
```

생성될 때까지

```bash
kubectl wait --for=create pod/<파드명> -n <네임스페이스> --timeout=30s
```

phase=Running

```bash
kubectl wait --for=jsonpath='{.status.phase}'=Running pod/<파드명> -n <네임스페이스> --timeout=60s
```

잡 완료

```bash
kubectl wait --for=condition=Complete job/<잡명> -n <네임스페이스> --timeout=180s
```

레이블 대상 전부 Ready

```bash
kubectl wait --for=condition=Ready pod -l <키>=<값> -n <네임스페이스> --timeout=60s
```

## kustomize

현재 디렉터리 빌드 (클러스터에 적용하지 않음)

```bash
kubectl kustomize
```

디렉터리 빌드

```bash
kubectl kustomize <디렉터리>
```

빌드 후 적용

```bash
kubectl apply -k <디렉터리>
```

빌드 후 삭제

```bash
kubectl delete -k <디렉터리>
```

Helm inflator 사용 빌드 (kustomization에 helmCharts가 있을 때)

```bash
kubectl kustomize --enable-helm <디렉터리>
```

## cp

로컬 → 파드 (컨테이너에 `tar` 필요)

```bash
kubectl cp <로컬경로> <네임스페이스>/<파드명>:<파드경로>
```

파드 → 로컬

```bash
kubectl cp <네임스페이스>/<파드명>:<파드경로> <로컬경로>
```

특정 컨테이너로 복사

```bash
kubectl cp <로컬경로> <네임스페이스>/<파드명>:<파드경로> -c <컨테이너명>
```

## auth can-i

파드 생성 권한

```bash
kubectl auth can-i create pods -n <네임스페이스>
```

모든 네임스페이스에서 파드 생성 가능한지

```bash
kubectl auth can-i create pods --all-namespaces
```

디플로이먼트 목록 권한

```bash
kubectl auth can-i list deployments.apps -n <네임스페이스>
```

시크릿 get 권한 (값은 조회하지 않음)

```bash
kubectl auth can-i get secrets -n <네임스페이스>
```

파드 로그 서브리소스

```bash
kubectl auth can-i get pods --subresource=log -n <네임스페이스>
```

현재 네임스페이스에서 전부 가능한지

```bash
kubectl auth can-i '*' '*'
```

허용된 동작 목록

```bash
kubectl auth can-i --list -n <네임스페이스>
```

종료 코드만 (스크립트용)

```bash
kubectl auth can-i delete pods -n <네임스페이스> -q
```

현재 사용자 정보

```bash
kubectl auth whoami
```

## 공통 플래그

네임스페이스 지정 (`-n`)

```bash
kubectl get pods -n <네임스페이스>
```

모든 네임스페이스 (`-A`)

```bash
kubectl get pods -A
```

YAML 출력

```bash
kubectl get deploy <디플로이명> -n <네임스페이스> -o yaml
```

JSON 출력

```bash
kubectl get deploy <디플로이명> -n <네임스페이스> -o json
```

추가 컬럼 (`-o wide`)

```bash
kubectl get pods -n <네임스페이스> -o wide
```

이름만 (`-o name`)

```bash
kubectl get pods -n <네임스페이스> -o name
```

변경 감시 (`-w`)

```bash
kubectl get pods -n <네임스페이스> -w
```

레이블 선택 (`-l`)

```bash
kubectl get pods -n <네임스페이스> -l <키>=<값>
```

파일/디렉터리 (`-f`)

```bash
kubectl apply -f <파일>
```

Kustomize (`-k`)

```bash
kubectl apply -k <디렉터리>
```

강제 삭제 (`--force` + `--grace-period`)

```bash
kubectl delete pod <파드명> -n <네임스페이스> --force --grace-period=0
```

클라이언트 드라이런

```bash
kubectl apply -f <파일> --dry-run=client
```

서버 드라이런

```bash
kubectl apply -f <파일> --dry-run=server
```

이번 명령만 컨텍스트

```bash
kubectl --context=<컨텍스트명> get nodes
```

이번 명령만 kubeconfig 경로

```bash
kubectl --kubeconfig=<kubeconfig경로> get nodes
```

전체 네임스페이스 + wide

```bash
kubectl get pods -A -o wide
```

## Helm / Argo CD / k3d

Helm 릴리스가 남긴 Secret 이름만 (값 출력 없음)

```bash
kubectl get secrets -n <네임스페이스> -l owner=helm
```

Argo CD Application

```bash
kubectl get applications -n <네임스페이스>
```

Argo CD Application YAML

```bash
kubectl get application <앱명> -n <네임스페이스> -o yaml
```

Argo CD AppProject

```bash
kubectl get appprojects -n <네임스페이스>
```

이름에 argo가 들어간 API 리소스

```bash
kubectl api-resources | grep -i argo
```

kube-system 파드 (k3d/Traefik/metrics)

```bash
kubectl get pods -n kube-system -o wide
```

기본 Traefik 서비스 (k3d)

```bash
kubectl get svc -n kube-system
```

리소스 직접 편집

```bash
kubectl edit deploy <디플로이명> -n <네임스페이스>
```

이미지 패치

```bash
kubectl set image deploy/<디플로이명> <컨테이너명>=<이미지> -n <네임스페이스>
```

레이블 추가

```bash
kubectl label pod <파드명> <키>=<값> -n <네임스페이스>
```

어노테이션 추가

```bash
kubectl annotate pod <파드명> <키>=<값> -n <네임스페이스>
```

디플로이먼트를 서비스로 노출

```bash
kubectl expose deploy <디플로이명> --port=<포트> --target-port=<포트> -n <네임스페이스>
```

일회성 파드 실행

```bash
kubectl run <파드명> --image=<이미지> -n <네임스페이스> --restart=Never
```

JSON 패치로 레플리카 변경

```bash
kubectl patch deploy <디플로이명> -n <네임스페이스> -p '{"spec":{"replicas":<개수>}}'
```

API 서버 헬스

```bash
kubectl get --raw=/healthz
```

셸 자동완성 (zsh)

```bash
kubectl completion zsh
```
