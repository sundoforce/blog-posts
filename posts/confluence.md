---
layout: post
title:  "confluence docker 설치"
subtitle: "confluence docker 설치"
author: "sdk(muse kim)"
catalog: true
header-mask:  0.3
tags:
-  confluence
-  docker
date:   2020-08-27
multilingual: flase
---

# 목차
1. docker 설치
2. native 설치
- [순서](#순서)
- [참고문서](#참고문서)
- [설치](#설치)


## 1. Docker 설치

### 순서

- docker설치
- docker postgresql
- docker conflucen
- 설정

쿠버네티스로 배포는 현재는 아무래도 어렵다. 먼저 docker를 설치 해보고 자 함.

## 참고 문서

**docker**

- confluence 설치 : [https://hub.docker.com/r/atlassian/confluence-server/](https://hub.docker.com/r/atlassian/confluence-server/)

- postgresql 설치 : [https://judo0179.tistory.com/48](https://judo0179.tistory.com/)

**일반설치**

- confluce 설치 : [https://muse.kim/844](https://muse.kim/844)

## 설치

### postgresql 설치

루트권한으로 설치 헀음.

```
# postgresql docker 이미지 다운
$ docker pull postgres

# docker볼륨 생성 후 실행
$ docker volume create pgdata
$ docker run -d -p 5432:5432 --name pgsql -it --rm -v pgdata:/var/lib/postgresql/data
# 도커 접속 후 계정 생성
$ docker exec -it pgsql bash
​
root@7f61651d217f:/# psql -U postgres
psql (10.3 (Debian 10.3-1.pgdg90+1))
Type "help" for help.
postgres=# CREATE DATABASE mytestdb;
CREATE DATABASE
postgres=#\q

# ip 확인
# 툴 설치
$ apt-get install net-tools
exit

$ docker exec 7f61651d217f ifconfig

```

### confluence 설치

```
$docker run -v /data/your-confluence-home:/var/atlassian/application-data/confluence --name="confluence" -d -p 8090:8090 -p 8091:8091 atlassian/confluence-server
```

### 접속

https://192.168.55.23:8091

![](https://sundoforce.github.io/assets/img/blog/2020-08-27-docker-confluce1.png)

- confluce 설치 : [https://muse.kim/844](https://muse.kim/844)

위 문서대로 설치 진행

### nginx

vi /etc/nginx/conf.d/domain.com.conf

```
server {
 listen 80;
 server_name domain.com;

location / {
 proxy_pass https://192.168.55.23:8091;
 }
}
```

### cloudflare

DNS설정 및 SSL설정 - 가변 설정


# 2.Native 설치 


Ubuntu 16.04 서버에 설치

## confluence 다운

https://ko.atlassian.com/software/confluence/download ![](/static/images/2018/confluence/스크린샷-2018-07-18-오전-9.28.13-965x1024.png)

설치 문서 https://confluence.atlassian.com/doc/installing-a-confluence-trial-838416249.html

## 파일업로드 후 설치파일 위치에서 명령어 실행

$ chmod a+x atlassian-confluence-X.X.X-x64.bin 권한업데이트

$ chmod a+x atlassian-confluence-6.10.1-x64.bin

설치파일 실행 $ sudo ./atlassian-confluence-X.X.X-x64.bin

$ sudo ./atlassian-confluence-6.10.1-x64.bin

글 작성 시기 : 2018년 7월 최신버전(confluence-6.10.1-x64)

log

Unpacking JRE ...
Starting Installer ...

This will install Confluence 6.10.1 on your computer.
OK [o, Enter], Cancel [c]
o
Click Next to continue, or Cancel to exit Setup.

Choose the appropriate installation or upgrade option.
Please choose one of the following:
Express Install (uses default settings) [1],
Custom Install (recommended for advanced users) [2, Enter],
Upgrade an existing Confluence installation [3]
1

See where Confluence will be installed and the settings that will be used.
Installation Directory: /opt/atlassian/confluence
Home Directory: /var/atlassian/application-data/confluence
HTTP Port: 8090
RMI Port: 8000
Install as service: Yes
Install [i, Enter], Exit [e]
i

Extracting files ...

Please wait a few moments while we configure Confluence.

Installation of Confluence 6.10.1 is complete
Start Confluence now?
Yes [y, Enter], No [n]
y

Please wait a few moments while Confluence starts up.
Launching Confluence ...

Installation of Confluence 6.10.1 is complete
Your installation of Confluence 6.10.1 is now ready and can be accessed via
your browser.
Confluence 6.10.1 can be accessed at http://localhost:8090
Finishing installation ...

## 설치 완료 후 설정

http://localhost:8090

![](/static/images/2018/confluence/스크린샷-2018-07-18-오전-10.15.01-1024x737.png)

![](/static/images/2018/confluence/스크린샷-2018-07-18-오전-10.15.27-1024x971.png)

![](/static/images/2018/confluence/스크린샷-2018-07-18-오전-10.39.25-1024x463.png)

![](/static/images/2018/confluence/스크린샷-2018-07-18-오전-10.39.54-1024x708.png)

## 한글꺠짐해결

출처 : https://www.chohi.ga/?p=93

폰트다운

sudo curl -o nanumfont.zip http://cdn.naver.com/naver/NanumFont/fontfiles/NanumFont_TTF_ALL.zip

압축해제

sudo unzip nanumfont.zip -d /opt/atlassian/confluence/jre/lib/fonts

/confluence/bin/setjre.sh 추가

CATALINA_OPTS="-Dconfluence.document.conversion.fontpath=/opt/atlassian/confluence/jre/lib/fonts ${CATALINA_OPTS}"
CATALINA_OPTS="-Dconfluence.document.conversion.words.defaultfontname=NanumGothic ${CATALINA_OPTS}"
CATALINA_OPTS="-Dconfluence.document.conversion.slides.defaultfontname.regular=NanumGothic ${CATALINA_OPTS}"
CATALINA_OPTS="-Dconfluence.document.conversion.slides.defaultfontname.asian=NanumGothic ${CATALINA_OPTS}"
CATALINA_OPTS="-Dconfluence.document.conversion.slides.defaultfontname.symbol=NanumGothic ${CATALINA_OPTS}"

재시동

## 도메인 연결

2018.07.31 (완료) https://confluence.sundo.kim https://doc.sundo.kim

http://confluence.goldpitcher.co.kr/pages/viewpage.action?pageId=142868487 https://confluence.atlassian.com/confeap/running-confluence-behind-nginx-with-ssl-849150880.html

https://chohi.ga(운영자님이 직접 도와주심)

nginx conf

```
server {
listen 80;
server_name doc.sundo.kim;

    return       301 https://$server_name$request_uri;

}

server {
listen 443 ssl http2;
server_name doc.sundo.kim;
ssl_certificate "/root/ssl/localhost.crt";
ssl_certificate_key "/root/ssl/localhost.key";

    ssl_session_timeout  5m;

    ssl_protocols  TLSv1.2;
    ssl_ciphers 'ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-CBC-SHA348:ECDHE-RSA-AES256-CBC-SHA:ECDHE-RSA-AES128-CBC-SHA256:ECDHE-RSA-AES128-CBC-SHA';
    ssl_prefer_server_ciphers   on;

    location / {
        client_max_body_size 100m;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://192.168.55.239:8090;
    }

}

```

현재 상황 ![](/static/images/2018/confluence/1111111.png)

## gmail 메일서버 발송 설정(안됨)

https://confluence.atlassian.com/confkb/how-do-i-configure-confluence-to-use-gmail-as-the-mail-server-42795030.html

gSuite는 유료임.

그래서 sub gmail  smtp 사용

![](/static/images/2018/confluence/메일-서버-보기-Confluence-사생활-보호-모드-2018-07-18-12-29-08-1024x442.png)

![](/static/images/2018/confluence/스크린샷-2018-07-18-오후-12.30.31-893x1024.png)

![](/static/images/2018/confluence/스크린샷-2018-07-20-오후-2.44.40-1024x420.png)

![](/static/images/2018/confluence/스크린샷-2018-07-20-오후-2.44.31-868x1024.png) ![](/static/images/2018/confluence/스크린샷-2018-07-20-오후-2.45.02-1024x826.png)
