---
layout: post
title:  "[k8s]jitsi 온라인 화상 프로그램 설치"
subtitle: "온라인 화상 프로그램 서버 설치"
author: "sdk(muse kim)"
catalog: true
header-mask:  0.3
tags:
- zoom
- jitisi
- webRTC
- kubernetes
date:  2020-07-14
multilingual: flase
---


# 목차

- [개요](#개요)
- [설치](#설치)
- [도메인](#도메인)
- [주저리](#주저리)

# 개요

`zoom` 처럼 직접 서버 구축해서 사용할 수 있는 오픈소스
쿠버네티스에는 배포 실패이고,

일단은 3가지 방법 시도 , 2가지 성공,

# 설치

## 1. 퀵설치

- 공식가이드 문서  
  [https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-quickstart](https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-quickstart)

```
# Run as root or with sudo

# Retrieve the latest package versions across all repositories
apt update

# Ensure support for apt repositories served via HTTPS
apt install apt-transport-https
```

- 설치

```
$sudo apt install jitsi-meet
```

자동설치됨,
중간에 도메인을 넣으면 SSL까지 해줌.

## 2. docker

docker-compose 설치

```
docker compose 설치
https://docs.docker.com/compose/install/

sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose


$ docker-compose --version


```

### git clone

공식문서  
[https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-docker](https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-docker)

```
$ git clone https://github.com/jitsi/docker-jitsi-meet && cd docker-jitsi-meet
```

- env 파일 복사

```
$cp env.example .env
```

- config 디렉토리에 생성

```
$mkdir -p ~/.jitsi-meet-cfg/{web/letsencrypt,transcripts,prosody/config,prosody/prosody-plugins-custom,jicofo,jvb,jigasi,jibri}
```

- docker 실행

```
$docker-compose up -d
```

접속  
https://192.168.55.71:8443

## 3. 메뉴얼 설치

지우는 방법

## 쿠버네티스 배포 ? install

- docker-jitsi-meet yaml 파일들 있음.

[https://github.com/jitsi/docker-jitsi-meet/tree/master/examples/kubernetes](https://github.com/jitsi/docker-jitsi-meet/tree/master/examples/kubernetes)

- 네임스페이스 생성

```
kubectl create namespace jitsi
```

- 아래 처럼 줄일 수 있음

`k create ns jitsi `

```
# Install guide for kubernetes

This guide will deploy jitsi in the most simple way: as several containers in a single pod. This is enough to start in case your hardware is enough. If you need to scale components to severa instance, you'll have to modify it to use several services and pods.

Create a namespace to deploy jitsi to:

`kubectl create namespace jitsi`

Add the secret with secret values (replace `...` with some random strings):

`kubectl create secret generic jitsi-config -n jitsi --from-literal=JICOFO_COMPONENT_SECRET=... --from-literal=JICOFO_AUTH_PASSWORD=... --from-literal=JVB_AUTH_PASSWORD=... `

Deploy the service to listen for JVB UDP traffic on all cluster nodes port 30300:

`kubectl create -f jvb-service.yaml`

Now we can deploy the rest of the application. First modify the `DOCKER_HOST_ADDRESS` env value in deployment.yaml to point to one of nodes in your cluster (or load-balancer for all nodes if you have one), and then deploy it:

`kubectl create -f deployment.yaml`

To expose the webapp, we can use Ingress (replace the `host` value with your actual hostname):

`kubectl create -f web-service.yaml`

You can either use "https" or "http" service port, depending on whether your ingress allows self-signed certs.
```

## 도메인

**NGINX**

```
proxy_pass http://localhost:8433;
```

```
server {
    listen 80;
    server_name  meet.domain.net;
    location / {
        proxy_set_header X-Forwarded-Host $host:$server_port;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass https://localhost:8443;
    }
}

```

- JITSI 커뮤니티
  [https://community.jitsi.org/](https://community.jitsi.org/)

## 주저리

쿠버네티스로 잘 구축해 보았으면...
