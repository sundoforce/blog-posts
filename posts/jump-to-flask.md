---
layout: post
title:  "[리뷰] 점프 투 플라스크"
subtitle: "Python Clone Project"
author: "sdk(muse kim)"
catalog: true
header-mask:  0.3
tags:
- python
- flask
date: 2021-03-01
multilingual: false
---


> `이지퍼블리싱` 출판사의 `출판사의 "점프 투 플라스크(박응용저)"`를 읽고 작성한 리뷰입니다.  
>  이 서평은 출판사의 `서적 제공`을 받아 작성하였습니다.  
>  최종 업데이트 : 2021-05-08

![2021-02-15-jump-to-flask-01](https://user-images.githubusercontent.com/16316626/138917025-ee2bdf8a-1eb9-4daf-8035-ece5c64d2e08.jpeg)

`점프 투 플라스크`는 `점프 투 파이썬`의 저자 박응용 선생님께서 집필하셨습니다.  
출간 기념 이벤트로 2020년 12월에 책을 받아 볼 수 있었습니다.

# 퇴근 후 2시간

### 퇴근 후 2시간, 한달이면 완성되는 홈페이지

퇴근 후 `2시간`, 제 기준 약 `2주일`이면  
python 과 flask로 게시판 홈페이지를 만들고  
`AWS(Amazon Web Service)` or `개인 서버`에 배포 할 수 있습니다.

[클론코딩 과정 기록(YouTube)](https://youtube.com/playlist?list=PLCNAgYWlpMrlUmA19fye3veXjXbmBnPLT)

책에 있는 소스코드를 보며 `error` 또는 `bug`가 발생할 경우  
저자 박응용 선생님의 홈페이지 [https://pybo.kr](https://pybo.kr)에 질문하실 수 있습니다.

그리고
전체 완성된 소스코드 **[https://github.com/pahkey/flaskbook](https://github.com/pahkey/flaskbook)**에
공개되어 있어 스스로 작성한 코드와 저자의 소스코드와 비교 해보며,  
어디가 잘못 되었는지 비교가능합니다.

## 최종 결과

[https://flask.qooo.io](https://flask.qooo.io])

책 내용에, 도메인 연결하는 방법도 있어 domain을 보유 하고 있다면,  
`pybo: (flask로 만든 app)`을 도메인에 연결 할 수 있습니다.

# 책 내용

> 종이책과 동일한 내용이 위키독스에 공개되어 있습니다.  
> [위키독스- 점푸 투 플라스크](https://wikidocs.net/book/4542)

`파이썬 설치부터 시작하여 서비스 운영까지 웹 프로그래밍의 처음부터 끝까지 구성`

- python 개발 환경 셋팅
    - 운영 및 개발 환경 분리
- AWS(Amazon Web Service)에 배포
- git 사용법
- 도메인 연결
- DB
    - sqlite
    - postgresql
    - ORM (Object Relational Mapping, 객체-관계 매핑)
- 최신트렌드 'bootstrap' 사용
- python + flask 게시판 형 홈페이지 (CRUD)

# Django VS FLASK

### FLASK

- 나는 Djanggo 프레임웍이 필요 없음.
- 게다가 종이책은 `박응용` 선생님의 책이 Djang보단 flask 도서가 빨리 나왔다.
- flask를 작은 restAPI 프레임워크라고 생각이 되었는데  
  `jump to flask`를 공부하다 보면 덤으로 bootstrap과 프론트부분도 근사하게 결과물이 나온다.
- flask로도 `파이썬 웹 개발부터 배포까지!` 할 수 있다.

### 부트스트랩 연결

이제 것 내가 접해본 flask는 restful Api로 생각하고
서버에서 처리한 결과를 http로 return 받거나 백엔드에서만 사용했었는데.
python에서 처리한 내용을 frontend에서 데이터를 받아서 처리가 되고
책을 따라하니 회원가입되는 게시판이 나왔다.

### 친절 한 소스코드

![2021-05-08-jump_to_flask](https://user-images.githubusercontent.com/16316626/138917044-810fdd4e-31ba-4dde-a890-4ed36308877a.jpeg)

나는 종종 실습 하다 보면 소스코드가 어느 경로에 위치하고 있는지,
어느 이름으로 저장할 지 헷갈릴 때가 많은데

- 기존 소스에 추가된 부분은 파랑색으로 나온다.
- 친절 하다: 많은 고민과 많은 배려를 느낄수 있었다.

소스코드가 한번에 있는것이 아니라 그때 그때 하나 하나 진도를 나가며
필요한것들을 구현하고 추가 하는 과정이 담겨 있다.
설명하는 풍선 박스가 도움이 된다.

### 따라서

한개 한개 기능을 추가하다보면
반복되는것들이 눈에 들어오고 감을 익힐 수 있었다

# 배포

이 책을 읽기 전에 나의 배포 방법은

### python run

```python
nohup flask run --host=0.0.0.0 > /home/flask/flask.log &
```

nohup으로 flask를 돌린다음 `Nginx`에서 proxy로 도메인을 연결 하였다.

### nginx 일부 코드

```nginx
server {
    location / {
        proxy_pass : host:port
       }
   }
```

## Gunicorn socket사용

> 출처 [https://wikidocs.net/81077](https://wikidocs.net/81077)

포트 소켓을 사용하기보단 Unix Socket을 활용하는 방법

- socket 등록
  `unix:/tmp/myproject.sock`
- flask 환경변수 `env`파일 추가
- Unix에 `service` 생성
- service 실행 등록

```nginx
location / {
            include proxy_params;
            proxy_pass http://unix:/tmp/myproject.sock;
    }
```

포트에서 -> socket으로 변경해서 사용

# 리뷰

책 리뷰는 처음 작성하는 것이라 어떻게 작성할지 막막하고  
잘쓰고 싶은 마음에 2020년 12월에 초안을 잡았지만  
더 이상 미루면 안될것 같아, 이 글을 마무리 지으려 합니다.

### 발전하고 소통 하는 책

- 위키독스와 파이보 홈페이지에 질문할수 있습니다.
- 피드백을 드리거나, 최신 내용은 위키독스에 업데이트 됩니다.

### 웹 개발의 A 부터 Z 까지

이 책은 python설치 부터, 환경변수, 웹개발, 배포, 디버깅까지  
전 과정을 한권에 책에 담은 책입니다.

`full stack`( `backend` + `frontend`) 개발자가 된 느낌을 누려 보세요.
도메인 연결, 개발환경과 운영환경 conf 분리 , 웹 서비스가 실제 local 어떻게 개발되고 어떻게 배포 되는  
노하우가 담겨 있습니다.

상용 cloud서비스인 `AWS`의 LightSail 에서 운영 방법까지 덤으로 익힐 수 있는 책입니다.

### 섬세함

페이징, 댓글 기능, 댓글의 추천기능, 댓글 작성 이후에 작성한 댓글 위치로 이동  
스크롤이 초기화 되는 문제 같은 지나치기 쉬운 부분들(귀찮은 부분)까지
실제로 운영이 가능한 그리고 저자님이 실제로 운영하고 계신 pybo.kr가 어떻게 만들어 졌는지
섬세함을 볼 수 있었습니다.

### ORM

아무래도 query문을 소스코드에 직접 작성하는것이 더 익숙하여,
ORM을 경험해 봐야겠다고 생각했습니다.
이 책에는 `join`, `outer join` , `subquery`도 ORM을 통해서  
어떻게 구현해야 하는지 설명 되어 있습니다.

## 마무리

책 한권으로 python, flask, frontend, db(sqlite, postgresql)의 기술을 모두 접해 볼수 있는 책입니다.  
python 기초 문법 책 `jump to python(박응용 저)`을 문법을 한번 보신 뒤,  
읽어 보시는것을 추천드립니다.

django 프레임웤을 다루는 Jump to django 책도 출판하셨다 합니다.

jump to python에 친필로 서명해 주신 박응용 선생님과
jump to flask를 제공해 주신 이지퍼블리싱께 출판사에게 감사드립니다.