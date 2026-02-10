---
layout: post
title: "[인프라] 도메인 구매부터 SSL 설치까지"
date: 2026-02-11 15:00:00 +0900
categories: ai
---

서버를 만들었으면, 이제 사람들이 접속할 수 있게 **도메인**을 붙이고, **안전한 연결(SSL)**을 설정해야 해. 이번엔 도메인 구매 → DNS 설정 → nginx 설정 → SSL 인증서 발급까지, 처음부터 끝까지 직접 해본 과정을 정리했어.

## 전체 흐름 한눈에 보기

```
도메인 구매 → DNS 설정 → nginx 설정 → SSL 발급
(이름 사기)   (이름표 붙이기) (안내원 배치) (봉투 씌우기)
```

## 1단계: 도메인 구매

### 도메인이 뭐야?

`3.36.19.0` 같은 IP 주소는 외우기 어려워. 그래서 **blitzchat.org** 같은 이름을 붙여주는 거야. 전화번호 대신 "엄마"라고 저장하는 것과 같아.

### 어디서 사?

도메인은 **등록기관(Registrar)**에서 사. 대표적으로:

- **Cloudflare Registrar** — 마진 없이 원가에 판매, DNS 관리가 편함
- **Namecheap** — 할인 많고 UI 깔끔
- **GoDaddy** — 가장 유명하지만 갱신 비용이 비쌈
- **가비아** — 한국 서비스, .kr 도메인에 좋음

나는 **Cloudflare Registrar**를 선택했어. 이유:

1. **원가 판매** — 마진을 안 붙여서 가장 쌈
2. **DNS 관리 통합** — 도메인 사고 바로 DNS 설정 가능
3. **자동 DNSSEC** — 보안 기능이 자동으로 켜짐

### 구매할 때 주의할 점

- **"사이트 추가(Add a Site)"와 "도메인 등록(Register Domain)"은 다르다**
  - "사이트 추가"는 다른 곳에서 산 도메인을 Cloudflare DNS로 관리하겠다는 것
  - "도메인 등록"이 진짜 도메인을 사는 것
- Cloudflare 대시보드 → **Domain Registration** → **Register Domains** 에서 구매
- `.com`이 가장 일반적이지만, 이미 누가 샀으면 `.org`, `.app`, `.io` 등 대안을 찾아야 해

## 2단계: DNS 설정

### DNS가 뭐야?

**DNS(Domain Name System)**는 인터넷의 전화번호부야. "blitzchat.org"를 입력하면 DNS가 "아, 그건 3.36.19.0이야"라고 알려줘.

### A 레코드 설정

DNS에는 여러 종류의 레코드가 있어. 가장 기본은 **A 레코드**야.

| 항목 | 값 |
|------|-----|
| Type | A |
| Name | @ (도메인 자체를 의미) |
| Content | 서버 IP 주소 (예: 3.36.19.0) |
| Proxy status | DNS only (회색 구름) |
| TTL | Auto |

**Proxy status 주의사항:**
- **주황 구름 (Proxied)** — Cloudflare가 중간에서 트래픽을 처리. 일반 웹사이트에 좋음
- **회색 구름 (DNS only)** — 직접 서버로 연결. **WebSocket은 반드시 이걸로 해야 해**

WebSocket은 클라이언트와 서버가 실시간으로 연결을 유지해야 해서, 중간에 프록시가 끼면 문제가 생길 수 있어.

### DNS 전파

레코드를 설정하고 바로 되는 게 아니야. **전파(Propagation)**라는 시간이 필요해.

전 세계 DNS 서버들이 "blitzchat.org = 3.36.19.0"이라는 정보를 공유하는 시간이야. 보통 **5~30분**, 최대 48시간까지 걸릴 수 있어.

확인하는 방법:

```bash
# Cloudflare DNS 서버에 물어보기
dig blitzchat.org +short @1.1.1.1

# Google DNS 서버에 물어보기
dig blitzchat.org +short @8.8.8.8

# 둘 다 서버 IP가 나오면 전파 완료!
```

## 3단계: nginx 리버스 프록시 설정

### 왜 nginx가 필요해?

우리 채팅 서버는 8082 포트에서 돌아가. 그런데 사용자는 443 포트(HTTPS)로 접속해. **nginx가 443으로 들어온 요청을 8082로 전달**해주는 거야.

```
사용자 → :443 (nginx) → :8082 (채팅 서버)
```

### 설정 파일

`/etc/nginx/sites-available/blitz` 파일을 만들어:

```nginx
upstream blitz_ws {
    server 127.0.0.1:8082;
}

server {
    listen 80;
    server_name blitzchat.org;

    location / {
        proxy_pass http://blitz_ws;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_read_timeout 60s;
        proxy_send_timeout 60s;
    }
}
```

핵심 설정 설명:

- **`upstream`** — 뒤에 있는 서버(채팅 서버)를 정의
- **`proxy_http_version 1.1`** — WebSocket은 HTTP/1.1이 필요해
- **`Upgrade`와 `Connection "upgrade"`** — HTTP 연결을 WebSocket으로 업그레이드하는 헤더. 이게 없으면 WebSocket이 안 돼
- **`proxy_read_timeout 60s`** — 60초 동안 데이터가 없으면 연결 끊음. 채팅 서버에서 ping/pong으로 유지해야 해

### 설정 적용

```bash
# 심볼릭 링크 생성 (sites-enabled에 등록)
sudo ln -sf /etc/nginx/sites-available/blitz /etc/nginx/sites-enabled/blitz

# 기본 설정 제거
sudo rm -f /etc/nginx/sites-enabled/default

# 문법 검사 & 적용
sudo nginx -t && sudo systemctl reload nginx
```

`nginx -t`에서 **"test is successful"**이 나와야 해. 에러가 나면 설정 파일에 오타가 있는 거야.

## 4단계: SSL 인증서 발급

### Let's Encrypt란?

**Let's Encrypt**는 **무료 SSL 인증서**를 발급해주는 기관이야. 예전에는 SSL 인증서를 사려면 돈을 내야 했는데, Let's Encrypt 덕분에 무료로 HTTPS를 쓸 수 있게 됐어.

단, 90일마다 갱신해야 해. 그래서 **certbot**이라는 도구가 자동으로 갱신해줘.

### certbot으로 발급

```bash
sudo certbot --nginx -d blitzchat.org --non-interactive --agree-tos --register-unsafely-without-email
```

옵션 설명:
- **`--nginx`** — nginx 설정을 자동으로 수정해서 HTTPS를 적용
- **`-d blitzchat.org`** — 이 도메인에 대한 인증서 발급
- **`--non-interactive`** — 질문 없이 자동 실행
- **`--register-unsafely-without-email`** — 이메일 없이 등록 (만료 알림을 못 받지만 간편)

이 명령어 하나로:
1. Let's Encrypt에서 인증서 발급
2. nginx 설정에 SSL 자동 추가 (443 포트, 인증서 경로)
3. HTTP → HTTPS 리다이렉트 설정

### 자동 갱신 테스트

```bash
sudo certbot renew --dry-run
```

**dry-run**은 실제로 갱신하진 않고 "갱신이 잘 될까?" 테스트만 하는 거야. 성공하면 90일마다 알아서 갱신돼.

## 완료 후 확인

모든 설정이 끝나면 이렇게 돼:

```
사용자 앱
  ↓
wss://blitzchat.org (도메인 + SSL)
  ↓
3.36.19.0:443 (nginx가 SSL 처리)
  ↓
127.0.0.1:8082 (채팅 서버)
```

- `wss://` — WebSocket + SSL (보안 WebSocket)
- `ws://` — 일반 WebSocket (보안 없음, 앱스토어 제출 불가)

## 삽질 기록

실제로 하면서 겪은 실수들:

1. **도메인 "추가"와 "구매"를 혼동** — Cloudflare에서 "Add a Site"을 눌러서 남의 도메인을 추가해버렸어. 이건 도메인을 사는 게 아니라, 이미 가지고 있는 도메인의 DNS를 관리하겠다는 거였어
2. **기존 DNS 레코드 정리** — 이전 소유자의 레코드가 남아있을 수 있어. 전부 삭제하고 새로 시작하는 게 깔끔해
3. **Lightsail 브라우저 콘솔에서 heredoc 안 됨** — 여러 줄 붙여넣기가 깨져. `nano`로 직접 편집하는 게 확실해
4. **nano에서 Ctrl 단축키 안 먹힘** — Lightsail 브라우저 콘솔이 Ctrl 키를 가로채. 화면을 직접 클릭한 후 키보드로 입력해야 해

---

도메인, DNS, nginx, SSL. 하나하나 뜯어보면 결국 "사용자가 안전하게 서버에 접속할 수 있게 만드는 과정"이야. 도메인은 기억하기 쉬운 이름, DNS는 이름을 주소로 바꿔주는 전화번호부, nginx는 요청을 안내하는 경비원, SSL은 통신을 보호하는 봉투. 이 네 가지를 연결하면 `wss://blitzchat.org`라는 한 줄로 전부 작동하게 돼.
