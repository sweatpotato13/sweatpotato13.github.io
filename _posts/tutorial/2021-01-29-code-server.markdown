---
layout: post
title: "code-server + Nginx + LetsEncrypt 구축하기"
date: "2021-01-29 23:03:44 +0900"
tags: [tutorial]
---

![ss](https://i.imgur.com/ufweHkZ.png)

기존에는 외부에서 가벼운 코딩을 하고자 할 때에는 아이패드로 jump desktop을 이용해서 집에 있는 맥미니에 원격을 붙여서 코딩을 하곤 했는데 원격 환경의 특성상 연결이 항상 깔끔하지 않아 많이 불편했는데 재밌는 레포지토리를 찾았습니다.

https://github.com/cdr/code-server

### 장점
* **깔끔한 환경**
* **기존 vscode와 놀랍도록 유사**
* **vscode의 확장 또한 사용 가능**

본 문서에선 code-server + Nginx에 HTTPS까지 적용하는 방법을 설명해 보곘습니다.

Mac OS를 기준으로 설명 되었습니다.

---

## code-server

<br>

### 1. code-server 설치
```bash
brew install code-server
```
![ss](https://i.imgur.com/UtIRFqQ.png)


brew가 설치되어 있지 않다면 brew를 먼저 설치해주세요 
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
<br>

### 2. code-server 설정 
```
nano ~/.config/code-server/config.yaml
```

code-server의 설정파일이 저장되어있는 파일입니다. 적절하게 설정값을 바꿔주세요

만약 해당 경로에 파일이 없다면 code-server를 실행하면 자동으로 생성됩니다.
```
brew services start code-server
```

<br>

### 3. code-server 재시작
```
brew services restart code-server
```

변경된 설정을 반영하기위해 code-server를 재시작합니다.
![ss](https://i.imgur.com/MfIi3ob.png)

<br>

### 4. code-server 접속
```
http://localhost:8080
```
![ss](https://i.imgur.com/32R4Fpz.png)
code-server의 기본포트는 8080 입니다.

<br>

---
## Nginx
code-server는 기본적으로 https 연결을 권장하기 때문에 Nginx를 이용하여 https환경 구축까지 해보겠습니다.

<br>

### 1. Nginx 설치
```
brew install nginx
```
![ss](https://i.imgur.com/Bpgytjw.png)

<br>

인텔맥과 M1맥의 Nginx 설치 경로가 다를 수 있어 본인의 디바이스의 Nginx 설치 경로를 파악하도록 합시다. 
보통 Nginx 설치시 path가 적혀있긴 합니다.

### 2. nginx.conf 설정 변경
```
http {
		...
		...
		...
    include servers/*;
}
```
저는 /opt/homebrew/etc/nginx/nginx.conf 경로에 nginx.conf 파일이 위치하였습니다.

해당 파일을 열어 http {..} 내부의 맨 아래 ```include server/*;``` 를 추가해줍니다. 이는 server 폴더 내의 다른 파일들을 불러와서 Nginx 설정으로 사용하겠다는 의미입니다.

<br>

### 3. code-server 컨픽 작성
```
server {
    listen 80;
    listen [::]:80;
    server_name mydomain.com;

    location / {
      proxy_pass http://localhost:8080/;
      proxy_set_header Host $host;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection upgrade;
      proxy_set_header Accept-Encoding gzip;
    }
}
```
저는 ```/opt/homebrew/etc/nginx/server/code-server``` 경로에 해당 파일을 생성하여 위 코드와 같은 컨픽을 작성하였습니다. 해당 컨픽은 code-server 공식 레포에도 동일하게 작성되어 있습니다.

해당 컨픽을 작성하고 Nginx를 재시작하면 해당 도메인의 80번포트로 접속하게 되면 proxy_pass로 연결됩니다.

---
## LetsEncrypt

마지막으로 https를 적용해 보겠습니다. 본 문서에서는 ssl 인증서를 무료로 발급해주는 LetsEncrypt를 사용합니다.

### 1. certbot 설치
```
brew install certbot
```

<br>

### 2. certbot을 이용하여 SSL 인증서 생성
```
sudo certbot --non-interactive --redirect --agree-tos --nginx --nginx-server-root /opt/homebrew/etc/nginx -d mydomain.com -m my@email.com
```

nginx-server-root 옵션은 본인의 nginx 경로에 맞게 수정하여 입력해줍니다.

위 커맨드를 수행하면 자동으로 인증서 생성부터 nginx https 적용까지 수행됩니다.

정상적으로 진행했다면 아래 사진과 같이 https 연결이 가능해집니다.

![ss](https://i.imgur.com/B9Ggy3v.png)

---


