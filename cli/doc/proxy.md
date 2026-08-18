# proxy

Ubuntu 노트북이 AWS 프록시로 `ssh -R` reverse SSH를 연다. 프록시는 그 터널(`127.0.0.1:<remote-port>`)로 노트북에 접속하고, `*.lab.origemite.com` 트래픽을 노트북(k3d)으로 넘긴다. 클러스터는 Mac에서 띄우지 않는다.

## Reverse SSH 터널 생성

노트북에서 로컬 SSH(22)를 프록시 `<remote-port>`에 역방향 연결한다.

```bash
ssh -N -R <remote-port>:127.0.0.1:22 <proxy-user>@<proxy-host>
```

노트북에서 HTTP(80)를 프록시 `<remote-port>`에 역방향 연결한다.

```bash
ssh -N -R <remote-port>:127.0.0.1:80 <proxy-user>@<proxy-host>
```

노트북에서 HTTPS(443)를 프록시 `<remote-port>`에 역방향 연결한다.

```bash
ssh -N -R <remote-port>:127.0.0.1:443 <proxy-user>@<proxy-host>
```

노트북에서 임의 로컬 포트를 프록시에 역방향 연결한다.

```bash
ssh -N -R <remote-port>:127.0.0.1:<local-port> <proxy-user>@<proxy-host>
```

노트북에서 80과 443을 한 세션에서 함께 연다.

```bash
ssh -N -R <http-remote-port>:127.0.0.1:80 -R <https-remote-port>:127.0.0.1:443 <proxy-user>@<proxy-host>
```

노트북에서 SSH·HTTP·HTTPS를 한 세션에서 함께 연다.

```bash
ssh -N -R <ssh-remote-port>:127.0.0.1:22 -R <http-remote-port>:127.0.0.1:80 -R <https-remote-port>:127.0.0.1:443 <proxy-user>@<proxy-host>
```

노트북에서 백그라운드로 터널만 연다.

```bash
ssh -fN -R <remote-port>:127.0.0.1:<local-port> <proxy-user>@<proxy-host>
```

노트북에서 포워딩 실패 시 바로 종료한다.

```bash
ssh -N -o ExitOnForwardFailure=yes -R <remote-port>:127.0.0.1:<local-port> <proxy-user>@<proxy-host>
```

노트북에서 keepalive와 함께 터널을 연다.

```bash
ssh -N -o ServerAliveInterval=30 -o ServerAliveCountMax=3 -o ExitOnForwardFailure=yes -R <remote-port>:127.0.0.1:<local-port> <proxy-user>@<proxy-host>
```

노트북에서 프록시 루프백에만 바인딩한다.

```bash
ssh -N -R 127.0.0.1:<remote-port>:127.0.0.1:<local-port> <proxy-user>@<proxy-host>
```

노트북에서 지정 키로 터널을 연다.

```bash
ssh -N -i ~/.ssh/<key-name> -R <remote-port>:127.0.0.1:<local-port> <proxy-user>@<proxy-host>
```

노트북에서 프록시 SSH 포트가 22가 아닐 때 연다.

```bash
ssh -N -p <ssh-port> -R <remote-port>:127.0.0.1:<local-port> <proxy-user>@<proxy-host>
```

노트북에서 verbose로 포워딩 성공 여부를 확인하며 연다.

```bash
ssh -vN -R <remote-port>:127.0.0.1:<local-port> <proxy-user>@<proxy-host>
```

## autossh 재연결

노트북에서 autossh로 터널을 유지한다.

```bash
autossh -M 0 -N -o ServerAliveInterval=30 -o ServerAliveCountMax=3 -o ExitOnForwardFailure=yes -R <remote-port>:127.0.0.1:<local-port> <proxy-user>@<proxy-host>
```

노트북에서 autossh로 80·443을 함께 유지한다.

```bash
autossh -M 0 -N -o ServerAliveInterval=30 -o ServerAliveCountMax=3 -o ExitOnForwardFailure=yes -R <http-remote-port>:127.0.0.1:80 -R <https-remote-port>:127.0.0.1:443 <proxy-user>@<proxy-host>
```

노트북에서 지정 키로 autossh를 연다.

```bash
autossh -M 0 -N -i ~/.ssh/<key-name> -o ServerAliveInterval=30 -o ServerAliveCountMax=3 -o ExitOnForwardFailure=yes -R <remote-port>:127.0.0.1:<local-port> <proxy-user>@<proxy-host>
```

노트북에서 백그라운드 autossh를 띄운다.

```bash
autossh -M 0 -f -N -o ServerAliveInterval=30 -o ServerAliveCountMax=3 -o ExitOnForwardFailure=yes -R <remote-port>:127.0.0.1:<local-port> <proxy-user>@<proxy-host>
```

실행 중인 autossh 프로세스를 찾는다.

```bash
pgrep -af autossh
```

`ssh -R` / autossh 관련 프로세스를 찾는다.

```bash
pgrep -af 'ssh .*-R|autossh'
```

autossh 설치 여부를 확인한다.

```bash
command -v autossh && autossh -V
```

## 터널 생존 확인

노트북에서 프록시로 가는 SSH 세션을 본다.

```bash
pgrep -af 'ssh .*<proxy-host>|ssh .*-R'
```

프록시에서 reverse 바인딩 포트를 본다.

```bash
ss -tlnp | grep '<remote-port>'
```

프록시에서 해당 포트로 TCP 연결이 되는지 본다.

```bash
nc -zv 127.0.0.1 <remote-port>
```

bash `/dev/tcp`로 프록시 루프백 포트를 확인한다.

```bash
timeout 3 bash -c 'echo > /dev/tcp/127.0.0.1/<remote-port>' && echo open || echo closed
```

노트북에서 로컬 대상 포트가 열려 있는지 본다.

```bash
ss -tlnp | grep -E ':(22|80|443|<local-port>)\b'
```

## 프록시에서 노트북 SSH

프록시에서 노트북으로 SSH한다.

```bash
ssh -p <remote-port> <laptop-user>@127.0.0.1
```

명령만 원격 실행한다.

```bash
ssh -p <remote-port> <laptop-user>@127.0.0.1 'hostname; uptime'
```

노트북 리슨 포트만 원격으로 확인한다.

```bash
ssh -p <remote-port> <laptop-user>@127.0.0.1 'ss -tlnp'
```

노트북 80/443만 원격으로 확인한다.

```bash
ssh -p <remote-port> <laptop-user>@127.0.0.1 'ss -tlnp | grep -E ":80|:443"'
```

## Mac에서 프록시 / 노트북

Mac에서 프록시로 SSH한다.

```bash
ssh <proxy-user>@<proxy-host>
```

Mac에서 프록시 SSH 포트가 22가 아닐 때 접속한다.

```bash
ssh -p <ssh-port> <proxy-user>@<proxy-host>
```

Mac에서 ProxyJump로 노트북에 붙는다.

```bash
ssh -o ProxyJump=<proxy-user>@<proxy-host> -p <remote-port> <laptop-user>@127.0.0.1
```

ProxyCommand로 노트북에 붙는다.

```bash
ssh -o ProxyCommand='ssh -W 127.0.0.1:<remote-port> <proxy-user>@<proxy-host>' <laptop-user>@127.0.0.1
```

프록시에 붙인 뒤 터널 포트로 노트북에 붙는다.

```bash
ssh -t <proxy-user>@<proxy-host> "ssh -p <remote-port> <laptop-user>@127.0.0.1"
```

## sshd / systemd

sshd가 활성인지 본다.

```bash
systemctl is-active ssh.service
```

sshd가 부팅 시 켜지는지 본다.

```bash
systemctl is-enabled ssh.service
```

sshd 상태를 본다.

```bash
systemctl status ssh.service --no-pager
```

sshd 설정을 테스트한다.

```bash
sudo sshd -t
```

포워딩 관련 sshd 설정을 읽기만 한다.

```bash
sudo sshd -T | grep -Ei 'gatewayports|allowtcpforwarding|permitlisten|permitopen|clientalive'
```

sshd_config에서 포워딩 키워드를 찾는다.

```bash
sudo grep -Ei 'GatewayPorts|AllowTcpForwarding|ClientAlive|ListenAddress|Port' /etc/ssh/sshd_config /etc/ssh/sshd_config.d/*.conf
```

커스텀 autossh 유닛 상태를 본다.

```bash
systemctl status <unit-name> --no-pager
```

커스텀 유닛을 재시작한다.

```bash
sudo systemctl restart <unit-name>
```

커스텀 유닛 부팅 등록을 본다.

```bash
systemctl is-enabled <unit-name>
```

사용자 systemd autossh 상태를 본다.

```bash
systemctl --user status <unit-name> --no-pager
```

로그아웃 후에도 사용자 서비스가 남는지 본다.

```bash
loginctl show-user <laptop-user> -p Linger
```

## 리슨 포트

TCP 리슨과 프로세스를 본다.

```bash
ss -tlnp
```

UDP 리슨도 포함해 본다.

```bash
ss -ulnp
```

22/80/443만 필터한다.

```bash
ss -tlnp | grep -E ':(22|80|443)\b'
```

숫자 포트 기준으로 소켓을 본다.

```bash
ss -tlnp sport = :<remote-port>
```

SSH 관련 소켓을 본다.

```bash
ss -antp | grep -i ssh
```

레거시 netstat로 리슨을 본다.

```bash
sudo netstat -tlnp
```

netstat에서 80/443만 본다.

```bash
sudo netstat -tlnp | grep -E ':(80|443)\b'
```

리슨 중인 TCP를 lsof로 본다.

```bash
sudo lsof -iTCP -sTCP:LISTEN
```

특정 포트를 lsof로 본다.

```bash
sudo lsof -i :<remote-port>
```

nginx가 잡은 포트를 본다.

```bash
sudo lsof -iTCP -sTCP:LISTEN | grep nginx
```

sshd가 잡은 포트를 본다.

```bash
sudo lsof -iTCP -sTCP:LISTEN | grep sshd
```

## nginx 설정 / sites-enabled

설정 문법만 검사한다.

```bash
sudo nginx -t
```

실제 로드된 전체 설정을 덤프한다.

```bash
sudo nginx -T
```

sites-enabled 목록을 본다.

```bash
ls -l /etc/nginx/sites-enabled/
```

sites-available 목록을 본다.

```bash
ls -l /etc/nginx/sites-available/
```

conf.d 목록을 본다.

```bash
ls -l /etc/nginx/conf.d/
```

stream 설정 디렉터리를 본다.

```bash
ls -l /etc/nginx/stream.d/ /etc/nginx/modules-enabled/
```

sites-available를 enabled에 연결한다.

```bash
sudo ln -s /etc/nginx/sites-available/<config-file> /etc/nginx/sites-enabled/<config-file>
```

enabled 링크를 제거한다.

```bash
sudo rm /etc/nginx/sites-enabled/<config-file>
```

와일드카드 관련 설정 파일을 찾는다.

```bash
sudo grep -Rni 'lab.origemite.com\|server_name\|proxy_pass\|listen' /etc/nginx/sites-enabled /etc/nginx/conf.d /etc/nginx/stream.d /etc/nginx/nginx.conf
```

nginx 버전을 본다.

```bash
nginx -v
```

stream 모듈 포함 여부를 본다.

```bash
nginx -V 2>&1 | tr ' ' '\n' | grep -i stream
```

로드된 설정에서 listen/server_name/proxy_pass만 뽑는다.

```bash
sudo nginx -T 2>/dev/null | grep -E 'server_name|listen|proxy_pass'
```

## nginx reload

문법 검사 후 reload한다.

```bash
sudo nginx -t && sudo systemctl reload nginx
```

nginx를 reload한다.

```bash
sudo systemctl reload nginx
```

nginx를 재시작한다.

```bash
sudo systemctl restart nginx
```

nginx 상태를 본다.

```bash
systemctl status nginx --no-pager
```

nginx가 활성·부팅등록인지 본다.

```bash
systemctl is-active nginx && systemctl is-enabled nginx
```

마스터에 reload 시그널을 보낸다.

```bash
sudo nginx -s reload
```

## nginx HTTP / stream

HTTP에서 `*.lab.origemite.com`을 터널로 넘긴다.

```nginx
server {
    listen 80;
    server_name *.lab.origemite.com;
    location / {
        proxy_pass http://127.0.0.1:<http-remote-port>;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

HTTPS를 nginx가 종료하지 않고 TCP로 넘긴다.

```nginx
stream {
    server {
        listen 443;
        proxy_pass 127.0.0.1:<https-remote-port>;
    }
}
```

HTTP도 stream으로 넘긴다.

```nginx
stream {
    server {
        listen 80;
        proxy_pass 127.0.0.1:<http-remote-port>;
    }
}
```

## curl 헬스체크

프록시 로컬 HTTP를 본다.

```bash
curl -I --max-time 5 http://127.0.0.1:<http-remote-port>/
```

프록시 로컬 HTTPS를 인증서 검증 없이 본다.

```bash
curl -Ik --max-time 5 https://127.0.0.1:<https-remote-port>/
```

노트북 로컬 80을 본다.

```bash
curl -I --max-time 5 http://127.0.0.1/
```

노트북 로컬 443을 본다.

```bash
curl -Ik --max-time 5 https://127.0.0.1/
```

와일드카드 도메인 HTTP를 본다.

```bash
curl -I --max-time 10 http://<host>.lab.origemite.com/
```

와일드카드 도메인 HTTPS를 본다.

```bash
curl -Ik --max-time 10 https://<host>.lab.origemite.com/
```

본문 없이 상태 코드와 시간을 본다.

```bash
curl -sS -o /dev/null -w '%{http_code} %{time_total}\n' --max-time 10 https://<host>.lab.origemite.com/
```

Host 헤더만 바꿔 프록시 로컬로 친다.

```bash
curl -I --max-time 5 -H 'Host: <host>.lab.origemite.com' http://127.0.0.1/
```

DNS를 고정해 프록시 IP로 직접 친다.

```bash
curl -Ik --max-time 10 --resolve <host>.lab.origemite.com:443:<proxy-ip> https://<host>.lab.origemite.com/
```

TLS 핸드셰이크만 본다.

```bash
echo | openssl s_client -connect <host>.lab.origemite.com:443 -servername <host>.lab.origemite.com
```

도메인 A 레코드를 본다.

```bash
dig +short <host>.lab.origemite.com
```

호스트명 해석을 본다.

```bash
getent hosts <host>.lab.origemite.com
```

## journalctl

sshd 최근 로그를 본다.

```bash
journalctl -u ssh.service -n 100 --no-pager
```

sshd를 따라간다.

```bash
journalctl -u ssh.service -f
```

nginx 최근 로그를 본다.

```bash
journalctl -u nginx.service -n 100 --no-pager
```

nginx를 따라간다.

```bash
journalctl -u nginx.service -f
```

커스텀 터널 유닛 로그를 본다.

```bash
journalctl -u <unit-name> -n 100 --no-pager
```

오늘 ssh/nginx만 본다.

```bash
journalctl -u ssh.service -u nginx.service --since today --no-pager
```

nginx 액세스 로그를 본다.

```bash
sudo tail -n 100 /var/log/nginx/access.log
```

nginx 에러 로그를 본다.

```bash
sudo tail -n 100 /var/log/nginx/error.log
```

auth 로그에서 ssh를 찾는다.

```bash
sudo grep -i ssh /var/log/auth.log | tail -n 50
```

## ufw / iptables

ufw 상태를 본다.

```bash
sudo ufw status verbose
```

ufw 규칙 번호를 본다.

```bash
sudo ufw status numbered
```

iptables 필터 테이블을 본다.

```bash
sudo iptables -L -n -v
```

NAT 테이블을 본다.

```bash
sudo iptables -t nat -L -n -v
```

nftables 규칙을 본다.

```bash
sudo nft list ruleset
```

80/443/22 관련 규칙만 본다.

```bash
sudo iptables -L -n -v | grep -E '22|80|443'
```

## scp / rsync

프록시에서 터널 경유 scp로 노트북에 보낸다.

```bash
scp -P <remote-port> <src-path> <laptop-user>@127.0.0.1:<dst-path>
```

프록시에서 터널 경유 scp로 노트북에서 가져온다.

```bash
scp -P <remote-port> <laptop-user>@127.0.0.1:<src-path> <dst-path>
```

Mac에서 프록시로 scp한다.

```bash
scp <src-path> <proxy-user>@<proxy-host>:<dst-path>
```

Mac에서 ProxyJump로 노트북에 scp한다.

```bash
scp -o ProxyJump=<proxy-user>@<proxy-host> -P <remote-port> <src-path> <laptop-user>@127.0.0.1:<dst-path>
```

rsync를 터널 SSH로 보낸다.

```bash
rsync -avz -e 'ssh -p <remote-port>' <src-path> <laptop-user>@127.0.0.1:<dst-path>
```

Mac에서 ProxyJump rsync로 노트북에 보낸다.

```bash
rsync -avz -e 'ssh -o ProxyJump=<proxy-user>@<proxy-host> -p <remote-port>' <src-path> <laptop-user>@127.0.0.1:<dst-path>
```

전송 없이 디렉터리만 비교한다.

```bash
rsync -avzn -e 'ssh -p <remote-port>' <src-path> <laptop-user>@127.0.0.1:<dst-path>
```

## ssh-keygen / ssh-copy-id

ed25519 키 쌍을 만든다.

```bash
ssh-keygen -t ed25519 -f ~/.ssh/<key-name> -C '<comment>'
```

공개키만 출력한다.

```bash
cat ~/.ssh/<key-name>.pub
```

공개키 지문을 본다.

```bash
ssh-keygen -lf ~/.ssh/<key-name>.pub
```

에이전트에 로드된 지문만 본다.

```bash
ssh-add -l
```

프록시에 공개키를 등록한다.

```bash
ssh-copy-id -i ~/.ssh/<key-name>.pub <proxy-user>@<proxy-host>
```

터널 경유로 노트북에 공개키를 등록한다.

```bash
ssh-copy-id -i ~/.ssh/<key-name>.pub -p <remote-port> <laptop-user>@127.0.0.1
```

authorized_keys 권한을 확인한다.

```bash
ls -l ~/.ssh/authorized_keys
```

ssh 디렉터리와 authorized_keys 권한을 맞춘다.

```bash
chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys
```

알려진 호스트 키 지문만 본다.

```bash
ssh-keygen -F <proxy-host>
```

## tmux / screen

노트북에서 tmux 세션을 만들어 터널을 띄운다.

```bash
tmux new -s <session-name> 'ssh -N -o ServerAliveInterval=30 -o ServerAliveCountMax=3 -o ExitOnForwardFailure=yes -R <remote-port>:127.0.0.1:<local-port> <proxy-user>@<proxy-host>'
```

노트북에서 분리된 tmux로 autossh를 띄운다.

```bash
tmux new -d -s <session-name> 'autossh -M 0 -N -o ServerAliveInterval=30 -o ServerAliveCountMax=3 -o ExitOnForwardFailure=yes -R <remote-port>:127.0.0.1:<local-port> <proxy-user>@<proxy-host>'
```

tmux 세션 목록을 본다.

```bash
tmux ls
```

tmux 세션에 다시 붙는다.

```bash
tmux attach -t <session-name>
```

노트북에서 screen으로 터널을 띄운다.

```bash
screen -S <session-name> ssh -N -o ServerAliveInterval=30 -o ServerAliveCountMax=3 -R <remote-port>:127.0.0.1:<local-port> <proxy-user>@<proxy-host>
```

노트북에서 분리된 screen으로 터널을 띄운다.

```bash
screen -dmS <session-name> ssh -N -o ServerAliveInterval=30 -o ServerAliveCountMax=3 -R <remote-port>:127.0.0.1:<local-port> <proxy-user>@<proxy-host>
```

screen 목록을 본다.

```bash
screen -ls
```

screen에 다시 붙는다.

```bash
screen -r <session-name>
```

## 노트북 k3d / ingress 포트

80/443 리슨을 본다.

```bash
ss -tlnp | grep -E ':(80|443)\b'
```

k3d 클러스터 목록을 본다.

```bash
k3d cluster list
```

k3d가 만든 컨테이너/포트를 본다.

```bash
docker ps --format 'table {{.Names}}\t{{.Ports}}\t{{.Status}}'
```

로드밸런서/인그레스 관련 서비스를 본다.

```bash
kubectl get svc -A | grep -Ei 'loadbalancer|ingress|traefik'
```

인그레스 목록을 본다.

```bash
kubectl get ingress -A
```

인그레스 컨트롤러 파드를 본다.

```bash
kubectl get pods -A | grep -Ei 'ingress|traefik|nginx'
```

노트북에서 로컬 80을 친다.

```bash
curl -I --max-time 5 http://127.0.0.1/
```

노트북에서 로컬 443을 친다.

```bash
curl -Ik --max-time 5 https://127.0.0.1/
```

Host 헤더로 인그레스 가상 호스트를 친다.

```bash
curl -I --max-time 5 -H 'Host: <host>.lab.origemite.com' http://127.0.0.1/
```

k3d 클러스터 상세를 본다.

```bash
k3d cluster get <cluster-name>
```

## SSH 연결 테스트

연결만 테스트한다.

```bash
ssh -o BatchMode=yes -o ConnectTimeout=5 <proxy-user>@<proxy-host> true
```

터널 경유 노트북 연결만 테스트한다.

```bash
ssh -o BatchMode=yes -o ConnectTimeout=5 -p <remote-port> <laptop-user>@127.0.0.1 true
```

멀티플렉스 소켓이 살아 있는지 본다.

```bash
ssh -O check -o ControlPath='~/.ssh/cm-%r@%h:%p' <proxy-user>@<proxy-host>
```

열린 마스터 연결을 닫는다.

```bash
ssh -O exit -o ControlPath='~/.ssh/cm-%r@%h:%p' <proxy-user>@<proxy-host>
```

## 빠른 진단

프록시에서 터널·nginx·포트를 한 번에 본다.

```bash
systemctl is-active ssh nginx; ss -tlnp | grep -E ':(22|80|443|<remote-port>)\b'; pgrep -af ssh
```

노트북에서 터널·80/443을 한 번에 본다.

```bash
pgrep -af 'ssh .*-R|autossh'; ss -tlnp | grep -E ':(22|80|443)\b'
```

공인 도메인과 로컬을 비교해 친다.

```bash
curl -sS -o /dev/null -w 'local:%{http_code}\n' --max-time 5 http://127.0.0.1/; curl -sS -o /dev/null -w 'domain:%{http_code}\n' --max-time 10 http://<host>.lab.origemite.com/
```
