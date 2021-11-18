---
layout: post
title:  "(초안)Grafana 다국어 커스터마이징"
subtitle: "그라파나 커스터마이징"
author: "sdk(sun-do,kim)"
catalog: true
header-mask:  0.3
tags:
- grafana
- Go
- 한국어
- angular-translate

date: 2021-11-12
multilingual: false 
---
# 0.목차

# 이유
그라파나는 ~~~ 입니다.  
2019년 참여한 프로젝트에서 그라파나를 사용할 기회가 있었습니다.

제가 맡은 역할은 설치와 포털서비스 연동 및 커스터마이징 작업을 하였습니다.   
그 중 한개 그라파나 한글화가 가능할까? 라는 가능성을 알아보는 과정과 테스트까지의 글입니다.  
당시 생각을 더듬어 보며 재현해보려고 합니다.     

이 글은 따라하면 그라파나가 실행되고, 설정한 언어가 나오는 튜토리얼 정도로 생각하시면 될 것 같습니다.


## 그라파나에서는 공식적으로 다국어를 지원하지 않습니다.
주소: git 이슈
그라파나 이슈에 수많은 댓글이 달려있고, 저도 댓글을 달았고,  가끔 누군가가 추가로 '우리 나라 언어도 지원해 주세요' 하는 댓글이 달릴때마다,  
메일이 오기 때문에 잊을만 하면 생각나는 이슈입니다.  
 
stackoverflow 에 찾아보면 `angular-translate`를 이용하면 된다는 댓글이 있습니다.
글 주소: 

# 그라파나 설치  
이 글에서는 github에서 clone을 받아서 네이티브로 설치해보겠습니다.

## 설치
### Go 환경변수 설정


## 빌드 방법 
```shell
go run build.go build  
go run build.go setup
```

go env



### 구동 및 정지 
nohup 으로 구동 및  
$ `go run grafana` 으로 해주면 됩니다.  
쉘 스크립트를 작성하였습니다.  

#### 구동 : start.sh
```shell
#!/bin/bash

ps -ef|grep grafana-server|grep -v grep|awk '{print $2}'|xargs kill -9 2> /dev/null
cd /home/odp/usr/local/go/src/github.com/grafana/grafana/bin/
nohup /home/odp/usr/local/go/src/github.com/grafana/grafana/bin/grafana-server &> /dev/null &
```
#### 정지: stop.sh 
```shell
#!/bin/bash
ps -ef|grep grafana-server|grep -v grep|awk '{print $2}'|xargs kill -9
```

시작과 종료는 쉘파일을 실행해 주세요.  
$`sh start.sh`
$`sh stop.sh`

# 수정해야 할 파일
1. TS
2. ss
3. 44 
4. ㅇㅇ

**1.ㄴㅇㄹㅇㄹ**
```go

```


# 결과 확인 

# 결론 
추가적으로 사용자가 설정한 언어의 정보를 저장하고,    
로그인했을때 설정파일을 읽어서 하는 작업도 필요해 보입니다.  

그라파나에서 다국어를 공식적으로 지원해 주면 가장 좋겠지만,    
이 테스트를 통해서 가능하다는 결론을 얻었습니다.  

덕분에 `Go`로 만들어진 `Grafana`도 접하게 되고, `TypeScript`도 구경해 보고 


## 부록 
### Kubernetes 에서 Grafana 


### 다른 작업들 
이 글에 포함하지 않은 커스터마이징은
포탈 사이트에 사용자가 가입하면 해당 사용자에 조직그룹에 메트릭정보를 표시하는 대시보드가 생성되게 하는 작업을 하였습니다.      
로그인하게 되면 사용자 권한 별로 그룹과 대시보드 조회하게 커스터마이징도 하였습니다.

> 예)  
> `플랫폼 관지자` 로그인: 모든 조직의 그룹과, 모든 사용자의 대시보드들을 볼수 있음  
> `조직관리자` 로그인: 본인 조직의 그룹과, 본인 조직의 모든 사용자의 대시보드들을 볼수 있음.    
> `일반 사용자` 로그인: 본인 대시보드 몇가지 종류만 확인 가능 함.


포털서비스와 그라파나의 SSO 는 JWT 기반인 keycloak 이라는 Open Source 를 사용했습니다.   

