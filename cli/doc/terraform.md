# terraform

## 전역 옵션

CLI 버전을 확인한다.
```bash
terraform version
```

버전을 JSON으로 확인한다.
```bash
terraform version -json
```

전체 도움말을 본다.
```bash
terraform -help
```

하위명령 도움말을 본다.
```bash
terraform <하위명령> -help
```

다른 디렉터리의 루트 모듈에서 명령을 실행한다.
```bash
terraform -chdir=<디렉터리> <하위명령>
```

변수 하나와 tfvars를 넣고 플랜을 파일로 저장한다.
```bash
terraform -chdir=<디렉터리> plan -var '<변수이름>=<값>' -var-file=<tfvars파일> -compact-warnings -out=<플랜파일>
```

입력 없이 플랜을 만든다.
```bash
terraform plan -input=false
```

색 없이 플랜을 만든다.
```bash
terraform plan -no-color
```

상태 잠금 대기 시간을 지정한다.
```bash
terraform plan -lock-timeout=<기간>
```

동시 작업 수를 제한한다.
```bash
terraform apply -parallelism=<숫자>
```

## init

작업 디렉터리를 초기화한다.
```bash
terraform init
```

다른 디렉터리를 초기화한다.
```bash
terraform -chdir=<디렉터리> init
```

프로바이더와 모듈을 제약 범위 안 최신으로 올린다.
```bash
terraform init -upgrade
```

백엔드 없이 초기화한다.
```bash
terraform init -backend=false
```

기존 상태를 옮기지 않고 백엔드를 다시 설정한다.
```bash
terraform init -reconfigure
```

백엔드 변경 시 상태를 이전한다.
```bash
terraform init -migrate-state
```

상태 이전 확인을 모두 수락한다.
```bash
terraform init -force-copy
```

입력 없이 초기화한다.
```bash
terraform init -input=false
```

자식 모듈 설치를 건너뛴다.
```bash
terraform init -get=false
```

로컬 플러그인 디렉터리만 사용한다.
```bash
terraform init -plugin-dir=<디렉터리>
```

lockfile을 바꾸지 않고 검증한다.
```bash
terraform init -lockfile=readonly
```

초기화 결과를 JSON으로 출력한다.
```bash
terraform init -json
```

자식 모듈만 가져온다.
```bash
terraform get
```

자식 모듈을 최신으로 다시 가져온다.
```bash
terraform get -update
```

## validate

구성 문법과 내부 일관성을 검사한다.
```bash
terraform validate
```

검사 결과를 JSON으로 출력한다.
```bash
terraform validate -json
```

색 없이 검사한다.
```bash
terraform validate -no-color
```

변수를 넣고 검사한다.
```bash
terraform validate -var '<변수이름>=<값>'
```

tfvars를 넣고 검사한다.
```bash
terraform validate -var-file=<tfvars파일>
```

다른 디렉터리를 검사한다.
```bash
terraform -chdir=<디렉터리> validate
```

## fmt

현재 디렉터리 구성을 표준 스타일로 맞춘다.
```bash
terraform fmt
```

하위 디렉터리까지 포맷한다.
```bash
terraform fmt -recursive
```

포맷이 맞는지 검사만 한다.
```bash
terraform fmt -check
```

재귀적으로 포맷 여부를 검사한다.
```bash
terraform fmt -check -recursive
```

포맷 변경 diff를 보여 준다.
```bash
terraform fmt -diff
```

파일에 쓰지 않고 확인만 한다.
```bash
terraform fmt -write=false
```

특정 디렉터리를 포맷한다.
```bash
terraform fmt <디렉터리>
```

특정 파일을 포맷한다.
```bash
terraform fmt <파일>
```

바꾼 파일 목록을 숨긴다.
```bash
terraform fmt -list=false
```

## plan

변경을 미리 본다.
```bash
terraform plan
```

플랜을 파일로 저장한다.
```bash
terraform plan -out=<플랜파일>
```

변수를 넣고 플랜한다.
```bash
terraform plan -var '<변수이름>=<값>'
```

tfvars를 넣고 플랜한다.
```bash
terraform plan -var-file=<tfvars파일>
```

tfvars와 변수를 넣고 플랜을 저장한다.
```bash
terraform plan -var-file=<tfvars파일> -var '<변수이름>=<값>' -out=<플랜파일>
```

경고를 짧게 보여 주며 플랜한다.
```bash
terraform plan -compact-warnings
```

원격 조회 없이 플랜한다.
```bash
terraform plan -refresh=false
```

인프라는 바꾸지 않고 상태만 맞추는 플랜을 만든다.
```bash
terraform plan -refresh-only
```

전체 삭제 플랜을 만든다.
```bash
terraform plan -destroy
```

삭제 플랜을 파일로 저장한다.
```bash
terraform plan -destroy -out=<플랜파일>
```

특정 리소스만 플랜한다.
```bash
terraform plan -target=<리소스주소>
```

여러 리소스만 플랜한다.
```bash
terraform plan -target=<리소스주소> -target=<리소스주소>
```

리소스 교체 플랜을 만든다.
```bash
terraform plan -replace=<리소스주소>
```

CI에서 쓸 종료 코드로 플랜한다.
```bash
terraform plan -detailed-exitcode
```

플랜 로그를 JSON으로 출력한다.
```bash
terraform plan -json
```

다른 디렉터리에서 플랜을 저장한다.
```bash
terraform -chdir=<디렉터리> plan -out=<플랜파일>
```

import 블록용 구성 초안을 만든다.
```bash
terraform plan -generate-config-out=<파일>
```

## apply

계획을 만들고 확인 후 적용한다.
```bash
terraform apply
```

확인 없이 적용한다.
```bash
terraform apply -auto-approve
```

저장해 둔 플랜을 적용한다.
```bash
terraform apply <플랜파일>
```

변수를 넣고 적용한다.
```bash
terraform apply -var '<변수이름>=<값>'
```

tfvars를 넣고 적용한다.
```bash
terraform apply -var-file=<tfvars파일>
```

확인 없이 tfvars로 적용한다.
```bash
terraform apply -auto-approve -var-file=<tfvars파일>
```

경고를 짧게 하고 확인 없이 적용한다.
```bash
terraform apply -compact-warnings -auto-approve
```

입력 없이 확인도 생략하고 적용한다.
```bash
terraform apply -input=false -auto-approve
```

원격 조회 없이 적용한다.
```bash
terraform apply -refresh=false
```

인프라는 바꾸지 않고 상태만 갱신한다.
```bash
terraform apply -refresh-only
```

확인 없이 상태만 갱신한다.
```bash
terraform apply -refresh-only -auto-approve
```

전체 삭제 모드로 적용한다.
```bash
terraform apply -destroy
```

리소스를 교체한다.
```bash
terraform apply -replace=<리소스주소>
```

확인 없이 리소스를 교체한다.
```bash
terraform apply -replace=<리소스주소> -auto-approve
```

여러 리소스를 교체한다.
```bash
terraform apply -replace=<리소스주소> -replace=<리소스주소>
```

다른 디렉터리에서 확인 없이 적용한다.
```bash
terraform -chdir=<디렉터리> apply -auto-approve
```

## destroy

관리 중인 리소스를 전부 삭제한다.
```bash
terraform destroy
```

확인 없이 전부 삭제한다.
```bash
terraform destroy -auto-approve
```

tfvars를 넣고 삭제한다.
```bash
terraform destroy -var-file=<tfvars파일>
```

확인 없이 tfvars로 삭제한다.
```bash
terraform destroy -auto-approve -var-file=<tfvars파일>
```

변수를 넣고 삭제한다.
```bash
terraform destroy -var '<변수이름>=<값>'
```

원격 조회 없이 빠르게 삭제한다.
```bash
terraform destroy -refresh=false
```

확인 없이 빠르게 삭제한다.
```bash
terraform destroy -refresh=false -auto-approve
```

다른 디렉터리의 리소스를 삭제한다.
```bash
terraform -chdir=<디렉터리> destroy
```

## workspace

현재 워크스페이스 이름을 본다.
```bash
terraform workspace show
```

워크스페이스 목록을 본다.
```bash
terraform workspace list
```

워크스페이스를 만들고 전환한다.
```bash
terraform workspace new <워크스페이스>
```

기존 워크스페이스로 전환한다.
```bash
terraform workspace select <워크스페이스>
```

워크스페이스를 삭제한다.
```bash
terraform workspace delete <워크스페이스>
```

비어 있지 않아도 워크스페이스를 강제 삭제한다.
```bash
terraform workspace delete -force <워크스페이스>
```

default로 돌아간 뒤 워크스페이스를 삭제한다.
```bash
terraform workspace select default
terraform workspace delete <워크스페이스>
```

다른 디렉터리의 워크스페이스 목록을 본다.
```bash
terraform -chdir=<디렉터리> workspace list
```

## state

상태에 있는 모든 리소스 주소를 본다.
```bash
terraform state list
```

주소로 상태 항목을 필터한다.
```bash
terraform state list <리소스주소>
```

원격 ID로 상태 항목을 찾는다.
```bash
terraform state list -id=<기존ID>
```

한 리소스의 상태 속성을 본다.
```bash
terraform state show <리소스주소>
```

현재 상태를 stdout으로 출력한다.
```bash
terraform state pull
```

현재 상태를 파일로 저장한다.
```bash
terraform state pull > <상태파일>
```

로컬 상태 파일을 현재 백엔드로 올린다.
```bash
terraform state push <상태파일>
```

검사를 무시하고 상태를 강제 푸시한다.
```bash
terraform state push -force <상태파일>
```

실객체는 남기고 상태에서만 제거한다.
```bash
terraform state rm <리소스주소>
```

여러 주소를 상태에서만 제거한다.
```bash
terraform state rm <리소스주소> <리소스주소>
```

상태 제거 대상을 미리 본다.
```bash
terraform state rm -dry-run <리소스주소>
```

상태 주소를 바꾸거나 모듈로 옮긴다.
```bash
terraform state mv <소스주소> <대상주소>
```

상태 이동을 미리 본다.
```bash
terraform state mv -dry-run <소스주소> <대상주소>
```

상태에 기록된 프로바이더 소스를 바꾼다.
```bash
terraform state replace-provider <FROM프로바이더> <TO프로바이더>
```

확인 없이 프로바이더 소스를 바꾼다.
```bash
terraform state replace-provider -auto-approve <FROM프로바이더> <TO프로바이더>
```

## output

루트 모듈 출력을 모두 본다.
```bash
terraform output
```

특정 출력을 본다.
```bash
terraform output <출력이름>
```

출력을 문자열로만 찍는다.
```bash
terraform output -raw <출력이름>
```

출력을 JSON으로 모두 본다.
```bash
terraform output -json
```

특정 출력을 JSON으로 본다.
```bash
terraform output -json <출력이름>
```

색 없이 출력을 본다.
```bash
terraform output -no-color
```

## show

현재 상태를 사람이 읽게 보여 준다.
```bash
terraform show
```

현재 상태를 JSON으로 보여 준다.
```bash
terraform show -json
```

저장한 플랜을 보여 준다.
```bash
terraform show <플랜파일>
```

저장한 플랜을 JSON으로 보여 준다.
```bash
terraform show -json <플랜파일>
```

색 없이 상태를 보여 준다.
```bash
terraform show -no-color
```

## providers

이 구성이 쓰는 프로바이더 트리를 본다.
```bash
terraform providers
```

프로바이더 스키마를 JSON으로 본다.
```bash
terraform providers schema -json
```

의존성 잠금 파일을 갱신한다.
```bash
terraform providers lock
```

Ubuntu 노트북용 플랫폼으로 잠근다.
```bash
terraform providers lock -platform=linux_amd64
```

Mac 편집과 Ubuntu 적용용으로 잠근다.
```bash
terraform providers lock -platform=linux_amd64 -platform=darwin_arm64
```

프로바이더를 로컬 미러로 복사한다.
```bash
terraform providers mirror <디렉터리>
```

## import

기존 객체를 Terraform 주소에 연결한다.
```bash
terraform import <리소스주소> <기존ID>
```

변수를 넣고 import한다.
```bash
terraform import -var '<변수이름>=<값>' <리소스주소> <기존ID>
```

tfvars를 넣고 import한다.
```bash
terraform import -var-file=<tfvars파일> <리소스주소> <기존ID>
```

입력 없이 import한다.
```bash
terraform import -input=false <리소스주소> <기존ID>
```

다른 디렉터리에서 import한다.
```bash
terraform -chdir=<디렉터리> import <리소스주소> <기존ID>
```

import 블록용 구성을 만든 뒤 적용한다.
```bash
terraform plan -generate-config-out=<파일>
terraform apply
```

## refresh

원격과 맞춰 상태만 갱신한다.
```bash
terraform refresh
```

tfvars를 넣고 상태를 갱신한다.
```bash
terraform refresh -var-file=<tfvars파일>
```

apply로 상태만 갱신한다.
```bash
terraform apply -refresh-only
```

확인 없이 apply로 상태만 갱신한다.
```bash
terraform apply -refresh-only -auto-approve
```

상태만 갱신하는 플랜을 본다.
```bash
terraform plan -refresh-only
```

## replace

해당 리소스를 교체로 플랜한다.
```bash
terraform plan -replace=<리소스주소>
```

해당 리소스를 교체 적용한다.
```bash
terraform apply -replace=<리소스주소>
```

확인 없이 교체한다.
```bash
terraform apply -replace=<리소스주소> -auto-approve
```

여러 리소스를 확인 없이 교체한다.
```bash
terraform apply -replace=<리소스주소> -replace=<리소스주소> -auto-approve
```

## target

한 리소스만 플랜한다.
```bash
terraform plan -target=<리소스주소>
```

한 리소스만 적용한다.
```bash
terraform apply -target=<리소스주소>
```

확인 없이 한 리소스만 적용한다.
```bash
terraform apply -target=<리소스주소> -auto-approve
```

여러 리소스만 적용한다.
```bash
terraform apply -target=<리소스주소> -target=<리소스주소>
```

한 리소스만 삭제한다.
```bash
terraform destroy -target=<리소스주소>
```

확인 없이 한 리소스만 삭제한다.
```bash
terraform destroy -target=<리소스주소> -auto-approve
```

tfvars를 넣고 한 리소스만 적용한다.
```bash
terraform apply -target=<리소스주소> -var-file=<tfvars파일>
```

한 리소스의 삭제 플랜을 저장한다.
```bash
terraform plan -destroy -target=<리소스주소> -out=<플랜파일>
```

## console

대화형 콘솔을 연다.
```bash
terraform console
```

tfvars를 넣고 콘솔을 연다.
```bash
terraform console -var-file=<tfvars파일>
```

변수를 넣고 콘솔을 연다.
```bash
terraform console -var '<변수이름>=<값>'
```

한 표현식을 평가하고 끝낸다.
```bash
echo '<표현식>' | terraform console
```

tfvars를 넣고 변수 값을 확인한다.
```bash
echo 'var.<변수이름>' | terraform console -var-file=<tfvars파일>
```

로컬 값을 확인한다.
```bash
echo 'local.<이름>' | terraform console
```

다른 디렉터리에서 콘솔을 연다.
```bash
terraform -chdir=<디렉터리> console
```

## force-unlock

원격 백엔드의 상태 잠금을 해제한다.
```bash
terraform force-unlock <락ID>
```

확인 없이 상태 잠금을 해제한다.
```bash
terraform force-unlock -force <락ID>
```

다른 디렉터리의 상태 잠금을 해제한다.
```bash
terraform -chdir=<디렉터리> force-unlock <락ID>
```

## graph

리소스 의존 그래프를 DOT로 출력한다.
```bash
terraform graph
```

plan 그래프를 출력한다.
```bash
terraform graph -type=plan
```

refresh-only 계획 그래프를 출력한다.
```bash
terraform graph -type=plan-refresh-only
```

destroy 계획 그래프를 출력한다.
```bash
terraform graph -type=plan-destroy
```

apply 그래프를 출력한다.
```bash
terraform graph -type=apply
```

저장한 플랜의 apply 그래프를 출력한다.
```bash
terraform graph -plan=<플랜파일>
```

순환 의존을 강조한다.
```bash
terraform graph -type=plan -draw-cycles
```

plan 그래프를 PNG로 저장한다.
```bash
terraform graph -type=plan | dot -Tpng > <파일>
```

plan 그래프를 SVG로 저장한다.
```bash
terraform graph -type=plan | dot -Tsvg > <파일>
```

tfvars를 넣고 그래프를 만든다.
```bash
terraform graph -type=plan -var-file=<tfvars파일>
```

## 랩 워크플로

디렉터리를 초기화하고 포맷·검사·플랜·적용·출력을 이어서 한다.
```bash
terraform -chdir=<디렉터리> init
terraform -chdir=<디렉터리> fmt -recursive
terraform -chdir=<디렉터리> validate
terraform -chdir=<디렉터리> plan -var-file=<tfvars파일> -compact-warnings -out=<플랜파일>
terraform -chdir=<디렉터리> show <플랜파일>
terraform -chdir=<디렉터리> apply <플랜파일>
terraform -chdir=<디렉터리> output
```

확인 없이 tfvars로 다시 적용한다.
```bash
terraform -chdir=<디렉터리> apply -auto-approve -var-file=<tfvars파일>
```

한 Helm 릴리스를 교체한다.
```bash
terraform -chdir=<디렉터리> apply -replace=helm_release.<이름> -auto-approve
```

랩 리소스를 정리한다.
```bash
terraform -chdir=<디렉터리> destroy -auto-approve
```

Mac과 Ubuntu에서 같은 프로바이더 버전을 잠근 뒤 초기화한다.
```bash
terraform providers lock -platform=linux_amd64 -platform=darwin_arm64
terraform init
```
