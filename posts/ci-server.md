---
layout: post
title:  "윈도우 CI Server 셋팅방법"
subtitle: " (Jenkins, nexus, maven)"
author: "sdk(muse kim)"
catalog: true
header-mask:  0.3
tags:
-  ci/cd
-  maven
- jenkins
date:   2017-11-28
multilingual: flase
---

젠킨스에 대한 설명과 설치 하는 방법에 관한 레퍼런스는 인터넷에 많이 있지만
전에 참여한 프로젝트에 써드파티 경험을 바탕으로 구축과 설정을 하면서 정리하고 싶어 글을 작성합니다.
운영체제 마다 설치방법이 다소 다를 수 있으며,
전자정부프레임워크와 windows의 환경에서 작성하였습니다.

`Jenkins`는 개발 구성원들의 작업내용을 통합, 자동 빌드, 배포해주는 편리한 Open Source CI 서버 입니다.

`nexus`는 라이브러리를 관라해주고 형상관리로부터 빌드 바이너리를 분리하고 빌드 바이너리를 배포해줍니다.

`maven`은 java 프로젝트 빌드, 테스트, 배포를 자동화 해주는 도구입니다.


![ci_1](https://user-images.githubusercontent.com/16316626/138558315-8ba14d03-94e0-4c11-8507-063bb87e3e36.png)

## 글의 순서

[1. 개발환경](#1.-개발환경)  
[2. 서버용 개발환경 설치](#2. 서버용 개발환경 설치)  
[3. Tomcat 설정](#3. Tomcat 설정)  
[4. Jenkins 설치 및 설정](#4.Jenkins-설치-및-설정)  
5. Nexus
6. PMD, FindBug, CheckStyle Plung-in

## 1.  개발환경