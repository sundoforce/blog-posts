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
-  jenkins
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

먼저 Continuous Integration Server (CI Server)에 관해  
잘 정리되어 있는 글을 링크합니다.  
[http://happystory.tistory.com/89](http://happystory.tistory.com/89)

글의 순서

1. 개발환경
2. 서버용 개발환경 설치
3. Tomcat 설정
4. Jenkins 설치 및 설정
5. Nexus
6. PMD, FindBug, CheckStyle Plung-in

## 1. 개발환경

버전은 작업할 당시의 최신버전 기준  
JAVA jdk1.8.0_131  
Jenkins jenkins-2.60.1  
nexus nexus-OSS 2.14.4-03

## 2. 서버용 개발환경 다운로드

2.1표준전자정부프레임쿼크 포탈에서 다운  
[https://www.egovframe.go.kr/](https://www.egovframe.go.kr/)

회원가입과 로그인이 필요한 서비스 입니다.

다운로드 -> 개발환경 -> 3.X다운로드 -> 서버용 개발환경 Vesrion3.5.0

※다운로드 게시판  
[다운로드 위치  
](https://www.egovframe.go.kr/EgovDevEnvRelease_300.jsp?menu=3&submenu=2)

2.2 다운받은 파일 압축 해제

1. eGoCI-3.5.0_64bit.exe 실행
2. 압축 경로 지정
3. 파일 확인

![](/static/images/ci-server/ci_1.png)

![](/static/images/ci-server/images/ci_2.png)

![](/static/images/ci-server/images/ci_3.png)

![](/static/images/ci-server/images/tomcat.png)

2.3 아파치 톰켓 8.5.16 다운로드 및 설치  
[https://tomcat.apache.org/download-80.cgi  
](https://tomcat.apache.org/download-80.cgi)

다운받은 파일을 아래의 경로에 위치 시킨다  
C:eGovCI-3.5.0_64bitbin

2.4 Nexus 최신버전 다운로드  
[http://www.sonatype.org/downloads/nexus-latest.war  
](http://www.sonatype.org/downloads/nexus-latest.war)

다운받은 war 파일을 아래의 경로에 위치 시킨다  
C:eGovCI-3.5.0_64bitbinapache-tomcat-8.5.16webapps

2.5 JDK 1.8.131 설치  
[http://www.oracle.com/technetwork/java/javase/downloads/index.html  
](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

1.8.131 버전을 다운 받은 후 아래의 경로에 위치  
C:eGovCI-3.5.0_64bitbin

2.6 Jenkins 최신 버전 다운  
젠키스 홈페이지  
[  
https://jenkins.io/download/](https://jenkins.io/download/)  
2.60.1 Generic Java package(.war)를 다운

아래의 경로에 위치  
C:eGovCI-3.5.0_64bitbinapache-tomcat-8.5.16webapps

※이제까지 다운받은 파일파일은  
eGovCI-3.5.0_64bit - bin - apache-tomcat-8.5.16 - webapps - nexus.war  
jenkins.war

## 3.톰켓설정

3.1 톰켓 실행파일(start.bat)수정  
C:eGovCI-3.5.0_64bitstart.bat

`set CATALINA_HOME=%CI_HOME%binapache-tomcat-7.0.62`  
을  
`set CATALINA_HOME=%CI_HOME%binapache-tomcat-8.5.16`

로 수정

· 젠킨스(JENKINS) 환경변수 추가  
`set JENKINS_HOME=%CI_HOME%homejenkins` 추가  
· 메이븐(mavne) 환경변수 추가  
`set MAVEN_HOME=%CI_HOME%binapache-maven-3.3.3`  
· 넥서스 (nexus) 환경변수 추가  
`set PLEXUS_NEXUS_WORK=%CI_HOME%homenexus`  
· 허드슨 삭제  
`set HUDSON_HOME=%CI_HOME%homehudson 삭제`  
· JDK 1.8.131로 수정  
`set JAVA_HOME=%CI_HOME%binjdk1.7.0_80` 을 `set JAVA_HOME=%CI_HOME%binjdk1.8.0_131`로 수정  
`set JRE_HOME=%CI_HOME%binjdk1.7.0_80jre` 을 `set JRE_HOME=%CI_HOME%binjdk1.8.0_131jre`로 수정  
`echo Using JENKINS_HOME: "%JENKINS_HOME%"`  
`echo Using PLEXUS_NEXUS_WORK: "%PLEXUS_NEXUS_WORK%"`

수정 전

수정 후

![](images/tomcat-stop.bat_.png)

3.2 톰켓 stop.bat파일 수정  
경로 : C:eGovCI-3.5.0_64bitstop.bat

JDK : jdk1.8.0_131 (다운받은 버전)으로 수정  
CATALLINA_HOME : apache-tomcat-8.5.16로 수정

수정 전

수정 후

3.3(옵션) 톰캣 포트 설정  
필수는 아니지만, 권장사항입니다.  
1)포트 설정

수정 파일 C:eGovCI-3.5.0_64bitbinapache-tomcat-8.5.16confserver.xml

Connector port="8080"부분을 원하는 포트로 변경가능하다.

![](images/port.png)

2. 사용자 추가 및 권한 롤 설정  
   수정파일 : C:eGovCI-3.5.0_64bitbinapache-tomcat-8.5.16conftomcat-users.xml

안에 롤과 아이디 비밀번호를 설정 후 저장  
예)

3. 로컬 이외의 외부에서 admin 접근 권한 설정  
   수정 파일 :  
   C:eGovCI-3.5.0_64bitbinapache-tomcat-8.5.16webappsmanagerMETA-INFcontext.xml

Valve 부분 주석

드디어 준비과정이 끝났다.

이제 톰캣을 구동해 보도록 하자

C:eGovCI-3.5.0_64bitstart.bat 파일을 더블 클릭하면

톰켓이 실행되고, Jenkins, nexus가 설치 된다.

※톰켓 실행확인  
http://127.0.0.1:port 또는  
http://localhost:port  
포트번호 생략 가능

![](images/tomcat-runing.png)

## 4. Jenkins 설치 및 설정

톰켓 구동이 확인 되면

4.1 jenkins 설치 페이지 접속

http://127.0.0.1:port/jenkins 혹은 http://localhost:port/jenkins에 접속

최초 접속 시 설치가 진행 되며, 본문에 안내 되어있는 경로

c:eGovCI-3.5.0_64bithomejenkinssecretsinitalAdminPassword

를 열어 패스워드를 복사 한뒤  
Administrator password에 입력

그리고[Continue]를 누른다.

![](images/jenkins-init-pw.png)

4.2 Customiz Jenkins

Install suggetsted plugins(권장 설정으로 설치 )를 선택하면 자동으로 플러그인이 다운되며, 설치가 진행된다,

※트러블 슈팅

Install Failures,

플러그인들이 완벽히 설치가 안될수도 있는데

플러그인들이 설치가 안되면, 나중에 문제가 생길 수 있다.

이때 [Retry]를 눌러 다시 진행한다.

![](images/retry.png)

4.3 플러그인이 정상적으로 설치되면 계정 입력 페이지가 나온다.

![](images/createUser.png)

정보 입력 후 [continue as admin] 눌러 설정페이지에 진입하도록 하자.

![](images/jenkins.png)

![](images/jenkins2.png)

5. 플러그인 설정

젠킨스가 설치 되면 플러그인 설정을 해주어야 한다.

![](images/4.png)

왼쪽메뉴에서 아래의 탭으로 이동한다.  
[Jenkins관리] -> [플러그인 관리] -> [설치가능 탭]

![](images/3.png)

메이븐 프로젝트를 위해서  
Maven관련 플러그인  
SSH 관련 플러그인을 설치해아 한다.

![](images/6.png)

-Maven 관련 (플러그인 검색 필터 키워드 : maven)  
Maven Artifact ChoiceListProvider (Nexus)  
Maven Integration plugin  
-SSH 관련 (플러그인 검색 필터 키워드 : SSH )  
Publish Over SSH  
SSh Slaves plugin

![](images/5.png)

![](images/123.png)

5.2 Jenkins Global Tool Configuration 설정  
[Jenkins 관리] - > [Global Tool Configuration]

Global Tool Configuration 진입 후  
5.2.1 JDK를 추가해야 한다.  
[Add JDK]  
JAVA_HOME  
C:eGovCI-3.5.0_64bitbin에있는 JDK 경로 입력  
(JDK 1.5 , 1.6, 1.7, 1.8)

5.2.2 Git 삭제  
(이글에 해당되는 프로젝트에는 git을 사용하지 않고 SVN을 사용하였다)

![](images/remove-git.png)

5.2.3 Ant 설정

[Add Ant] 버튼을 눌러  
C:eGovCI-3.5.0_64bitbin 경로에 있는 Ant 경로를 설정해준다.  
해당 프로젝트 버전  
1.9.5  
1.10.1

5.2.3 Maven 설정  
[Add Maven] 눌러 C:eGovCI-3.5.0_64bitbin 경로에 있는 Maven 경로 추가

버전 3.3.3 , 3.2.5 , 3.5.0

![](images/ssh.png)

5.3.4 SSH 설정  
메뉴 진입  
좌측 [Jenkins관리] -> [시스템관리]

HostName과 Usename은 서버 정보는 각 서버의 설정에 맞게 해준다.

![](images/ssh2.png)

6. Nexus 설정

http://localhost/nexus에 접속하여 Repositories를 추가한다.  
[Repositories] -> [add] -> [Proxy Repository]

![](images/nexus-2.png)

![](images/nexus-1.png)

6.PMD,FindBug, CheckStyle Plug-in 설치 및 설정 [선택]

PMD: 미사용 변수, 비어있는 코드 블락, 불필요한 오프젝트 생성과 같은 결함 (defect)을 유발 할 수 있는 코드를 검사

FindBugs

checkstyle

![](images/pmd.png)

![](images/findbug.png)

![](images/Chekstyle.png)

6.1 plug-in 설정

![](images/bild-stting.png)

bulid Setting에 CheckStyle, Find Bug, Pmd를 선택하면  
빌드 될때 정적 분석을 해준다.

7.1 사용자 권한 설정

부모 프로젝트가있고 자식 프로젝트가 있을 경우 혹은 계정별로 권한이 필요한 경우  
사용자를 추가하고 권한(Role)을 부여 할 수 있다.

![](images/user.png)

Jenkins 관리 -> Manage Users

마치며,

전자정부프레임웤 윈도우 환경에서 셋팅한 상황입니다.  
실제로 빌드와 배포가 되었지만

서버정보와 프로젝트등이 없음으로 각 서버와 프로젝트의 상황에 맞게 배포해야합니다.

빌드 및 배포는 시간별, 명령어로, 수동으로 지정 할 수 있습니다.

현재 저는 windows 내에서 jenkins를 구동하고 있지 않습니다.

추후 다른 운영체제에서 이번 글에서 못다룬 내용들을 작성하도록 하겠습니다.

## Ubuntu에서 jenkins 설치

공식적으로 Jenkins에서 제공하는 설치하는 문서를 참고  
https://wiki.jenkins.io/display/JENKINS/Installing+Jenkins+on+Ubuntu

가이드에 따라서

wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
