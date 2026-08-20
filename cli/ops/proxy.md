# proxy ops

실제 호스트·포트·유저가 들어간 복붙 명령만 둔다. 기본 reverse SSH·nginx는 [`cli/doc/proxy.md`](../doc/proxy.md).

## Phase 7 서브존 Host 라우팅

Phase 7 FQDN은 `demo.<controller>.lab.origemite.com` (3레이블). DNS 이름 목록(외부 Route53, 변경 없음)은 [`dns/inventory.yaml`](../../dns/inventory.yaml) / [`dns-subzone.md`](dns-subzone.md).

**부모 존** `*.lab.origemite.com` (`argocd.lab…` 등) → 기존 reverse SSH 터널 → 노트북 80/443 → 기본 ingress (.201 또는 k3d published 80/443).

**서브존** 7종 → 프록시 nginx가 `server_name`으로 분기 → 노트북 `127.0.0.1:820x` → MetalLB `172.18.255.20x:80`.

| Host | 프록시 upstream (터널) | 노트북 로컬 | MetalLB |
| --- | --- | --- | --- |
| demo.nginx.lab.origemite.com | 127.0.0.1:8201 | 8201 | 172.18.255.201 |
| demo.gateway.lab.origemite.com | 127.0.0.1:8202 | 8202 | 172.18.255.202 |
| demo.cilium.lab.origemite.com | 127.0.0.1:8203 | 8203 | 172.18.255.203 |
| demo.kong.lab.origemite.com | 127.0.0.1:8204 | 8204 | 172.18.255.204 |
| demo.traefik.lab.origemite.com | 127.0.0.1:8205 | 8205 | 172.18.255.205 |
| demo.istio.lab.origemite.com | 127.0.0.1:8206 | 8206 | 172.18.255.206 |
| demo.haproxy.lab.origemite.com | 127.0.0.1:8207 | 8207 | 172.18.255.207 |

### 프록시 nginx — Host별 upstream

`/etc/nginx/sites-available/phase7-subzones` 예시. `<http-remote-port>`는 기존 부모 터널(80) 포트.

```nginx
# 부모 존 — 기존
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

# Phase 7 서브존 — Host별 820x
server {
    listen 80;
    server_name demo.nginx.lab.origemite.com;
    location / {
        proxy_pass http://127.0.0.1:8201;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
server {
    listen 80;
    server_name demo.gateway.lab.origemite.com;
    location / { proxy_pass http://127.0.0.1:8202; proxy_set_header Host $host; }
}
server {
    listen 80;
    server_name demo.cilium.lab.origemite.com;
    location / { proxy_pass http://127.0.0.1:8203; proxy_set_header Host $host; }
}
server {
    listen 80;
    server_name demo.kong.lab.origemite.com;
    location / { proxy_pass http://127.0.0.1:8204; proxy_set_header Host $host; }
}
server {
    listen 80;
    server_name demo.traefik.lab.origemite.com;
    location / { proxy_pass http://127.0.0.1:8205; proxy_set_header Host $host; }
}
server {
    listen 80;
    server_name demo.istio.lab.origemite.com;
    location / { proxy_pass http://127.0.0.1:8206; proxy_set_header Host $host; }
}
server {
    listen 80;
    server_name demo.haproxy.lab.origemite.com;
    location / { proxy_pass http://127.0.0.1:8207; proxy_set_header Host $host; }
}
```

프록시에서 적용.

```bash
sudo ln -sf /etc/nginx/sites-available/phase7-subzones /etc/nginx/sites-enabled/phase7-subzones
sudo nginx -t && sudo systemctl reload nginx
```

프록시에서 8201–8207 터널 바인딩 확인.

```bash
ss -tlnp | grep -E ':(820[1-7])\b'
nc -zv 127.0.0.1 8201
```

### 노트북 — reverse SSH 다중 -R (8201–8207)

노트북에서 MetalLB 앞단 socat(아래)을 띄운 뒤, 프록시로 820x 터널을 연다.

```bash
autossh -M 0 -N \
  -o ServerAliveInterval=30 -o ServerAliveCountMax=3 -o ExitOnForwardFailure=yes \
  -R 8201:127.0.0.1:8201 \
  -R 8202:127.0.0.1:8202 \
  -R 8203:127.0.0.1:8203 \
  -R 8204:127.0.0.1:8204 \
  -R 8205:127.0.0.1:8205 \
  -R 8206:127.0.0.1:8206 \
  -R 8207:127.0.0.1:8207 \
  <proxy-user>@<proxy-host>
```

기존 80/443/22 터널과 **같은 SSH 세션**에 `-R`을 추가해도 된다.

### 노트북 — socat으로 820x → MetalLB

노트북에서 각 로컬 포트를 MetalLB IP:80으로 포워드.

```bash
socat TCP-LISTEN:8201,fork,reuseaddr TCP:172.18.255.201:80 &
socat TCP-LISTEN:8202,fork,reuseaddr TCP:172.18.255.202:80 &
socat TCP-LISTEN:8203,fork,reuseaddr TCP:172.18.255.203:80 &
socat TCP-LISTEN:8204,fork,reuseaddr TCP:172.18.255.204:80 &
socat TCP-LISTEN:8205,fork,reuseaddr TCP:172.18.255.205:80 &
socat TCP-LISTEN:8206,fork,reuseaddr TCP:172.18.255.206:80 &
socat TCP-LISTEN:8207,fork,reuseaddr TCP:172.18.255.207:80 &
```

socat 리슨 확인.

```bash
ss -tlnp | grep -E ':(820[1-7])\b'
```

### end-to-end 확인

```bash
curl -sS -o /dev/null -w '%{http_code}\n' http://demo.nginx.lab.origemite.com/
curl -sS -o /dev/null -w '%{http_code}\n' http://demo.istio.lab.origemite.com/
```

클러스터만 검증(MetaLB 직접).

```bash
curl -sS -H 'Host: demo.nginx.lab.origemite.com' http://172.18.255.201/
curl -sS -H 'Host: demo.istio.lab.origemite.com' http://172.18.255.206/
```
