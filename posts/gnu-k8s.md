---
title:  "그누보드 쿠버네티스 배포"
subtitle: "docker, kubernetes"
tags:
- gnuboard
- kubernetes
- k8s
date: 2021-05-01
image: https://user-images.githubusercontent.com/34048253/138473541-ca9eef72-8d1f-4a24-8115-fe96b01f1eb0.png
---

> 시도 해본 사항입니다.  
> 참고만 해주세요. 
# 그누보드 쿠버네티스 설치

이 글은 Ubuntu 18.04 LTS 운영체제에서 `kubernetes`환경에서  `LNMP` 소프트웨어를 설치하고 구성하는 방법에 대해 설명합니다.

**LNMP(Linux + Nginx + MariaDB + PHP-FPM)** 구성을
이 글에서는 Ubuntu 18.04 LTS 운영체제에 k8s로 설명합니다.

* `kubespray`로 `kubernetes`를 설치하시려면   
  [https://qooo.io/cloud/install-kubernetes/](https://qooo.io/cloud/image-deploy-k8s/)로 이동하세요.
* docker image로 k8s에 배포 하시려면   
  [https://qooo.io/cloud/image-deploy-k8s/](https://qooo.io/cloud/image-deploy-k8s/)로 이동하세요.


## 순서
* PVC
    * clone 그누보드
    * /data/ mount
* php-fpm
* Nginx ConfigMap 구성
* Deployment
    * nginx
    * php-fpm
* Ingress 설정
    * ssl: cert-manager.io 사용
    * secret 생성
    * domain 연결
    *

#### TODO
* DB(MariaBD) -> k8s (현재 다른 서버에 있음)
* k8s 백업 환경 구성 -> 별도 게시글로

### 최종 yaml 종류
* pod.yaml
* service.yaml
* deployment.yaml
* nginxConfig.yaml

### 쿠버네티스 기본 명령어
kubectl -> k

## Create Persistent Volume
###  PV(Persistent Volume)생성
> [https://kubernetes.io/ko/docs/concepts/storage/persistent-volumes/](https://kubernetes.io/ko/docs/concepts/storage/volumes/)  
> 퍼시스턴트 볼륨 생성  
> [https://bcho.tistory.com/1259](https://bcho.tistory.com/1259)    
> 쿠버네티스 #5 - 디스크 (볼륨/Volume) - 참고: 조대협의 블로그
> [Kubernetes Persistent Volume 생성하기 - PV, PVC](https://waspro.tistory.com/580)

**pv.yaml**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dir
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```
### 참고 PV 에제들
**NFS**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: wildfly-pv
  labels:
    name: wildfly-pv
spec:
  capacity:
    storage: 10Ti
  accessModes:
  - ReadWriteMany
  nfs:
    server: 192.168.56.104
    path: /mypath
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: wildfly-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
   requests:
    storage: 10Ti
  selector:
    matchLabels:
      name: wildfly-pv
```
**HostPath**
```yaml 
apiVersion: v1
kind: Pod
metadata:
  name: hostpath
spec:
  containers:
    - name: redis
      image: redis
      volumeMounts:
        - name: terrypath
          mountPath: /data/shared
  volumes:
    - name: terrypath
      hostPath:
        path: /tmp
        type: Directory
```

**`repoGit`** 지원 하지 않음.

## PHP-FPM 서비스 생성


**php_service.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: php
  labels:
    tier: backend
spec:
  selector:
    app: php
    tier: backend
  ports:
protocol: TCP
port: 9000 
```

## Create PHP-FPM Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php
  labels:
    tier: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: php
      tier: backend
  template:
    metadata:
      labels:
        app: php
        tier: backend
    spec:
      volumes:
      - name: dir
        persistentVolumeClaim:
          claimName: dir
      containers:
      - name: php
        image: woosungchoi/fpm-alpine:latest
        volumeMounts:
        - name: dir
          mountPath: /dir
      initContainers:
      - name: install
        image: busybox
        volumeMounts:
        - name: dir
          mountPath: /dir
        command:
        - git 
        - "clone"
        - https://github.com/gnuboard/gnuboard5
        - "/dir/"
```
image: `woosungchoi/fpm-alpine:latest`
> 우성짱님의 php-8.x 버전:  항상 최신 fpm으로 유지 가능하다.

### 결과
`k get svc`

```shell

```


PV에는 여러가지 종류가 있습니다.


그누보드 디렉토리 생성
```shell
mkdir 
```

## Ingress
### clone ingress
```shell
$ git clone https://github.com/kubernetes/ingress-nginx.git

Cloning into 'ingress-nginx'...
remote: Enumerating objects: 100500, done.
remote: Counting objects: 100% (149/149), done.
remote: Compressing objects: 100% (93/93), done.
remote: Total 100500 (delta 61), reused 110 (delta 45), pack-reused 100351
Receiving objects: 100% (100500/100500), 117.29 MiB | 10.61 MiB/s, done.
Resolving deltas: 100% (55540/55540), done.
```

### 네임스페이스 생성
```shell
kubectl create namespace ingress-nginx
```

### Ingress 확인
```shell
root@node1:~/kubernetes-book/ingress# kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.233.5.71     <none>        80:30326/TCP,443:30910/TCP   8m54s
ingress-nginx-controller-admission   ClusterIP   10.233.27.122   <none>        443/TCP                      8m54s
root@node1:~/kubernetes-book/ingress#
```


### 참조사이트
> [Kubernetes Basic PHP 7.3 Application with Nginx on Google Cloud](https://www.cloudbooklet.com/kubernetes-basic-php-application-with-nginx-on-google-cloud/)    
> [[초보]쿠버네티스에서 그누보드5 서비스 설정하기](https://sir.kr/g5_tip/13098) - 영통사람  
> [https://sir.kr/g5_tip/12697](https://sir.kr/g5_tip/12697) - 명량폐인  
> [도커(Docker)를 이용한 그누보드5 로컬 개발환경 설정](https://www.wsgvet.com/ubuntu/123) - 우성짱  
> [Ubuntu 18.04 LTS 에서 웹서버(NGINX + PHP-FPM + MariaDB) 구성하기](https://blog.lael.be/post/8319) - 라엘 
