---
layout: post
title:  "아마존 도메인 등록"
subtitle: "아마존 도메인 등록"
author: "sdk(muse kim)"
catalog: true
header-mask:  0.3
tags:
- aws
- amazon
- domain
date:   2017-12-26
multilingual: flase
---

> 현재는 AWS Domain 서비스를 사용하지 않습니다.

https://www.youtube.com/watch?v=yLUxIs05dSM

## 아마존으로 도메인을 옮기는 이유

1. 관리하는 도메인이 많아 졌습니다.
2. 안정적인 서비스를 받아 보고 싶습니다.
3. 국내 도메인 등록비용과 연장 비용이 비쌉니다.
4. 레코드 설정과, name server 등록이 간편합니다.

.kr 도메인의 경우는 국내 최저가 웹티즌에서 vip가격으로 서비스 받고 있습니다.

## 1. 아마존 aws 서비스에 로그인 합니다.

[https://aws.amazon.com/ko/](https://aws.amazon.com/ko/)

![](/static/images/aws-register-domain/스크린샷-2017-12-27-오후-11.27.09.png)

또는 아래의 링크를 클릭하시면 router 53 도메인 등록 서비스로 이동합니다. [https://console.aws.amazon.com/route53/home#DomainRegistration:](https://console.aws.amazon.com/route53/home#DomainRegistration:)

## 2. Router 53으로 이동

![](/static/images/aws-register-domain/스크린샷-2017-12-27-오후-11.27.46.png)

![](/static/images/aws-register-domain/Screenshot_2017-12-27-233028-GMT9-1024x770.png)

저는 어제 2개의 도메인을 등록했고 한개는 정상적으로 집에 있는 서버로 A 레코드를 입력하여 정상적으로 작동합니다.

한개는 1년 무료 체험인 EC2 서버에 연결 해보도록 하겠습니다.

## 3. Registered domains

![](/static/images/aws-register-domain/Screenshot_2017-12-27-233320-GMT9.png)

[Register Domain] 을 눌러 도메인 등록 화면으로 이동합니다.

옆에 있는 Transter Domain은 기존의 등록되어있는 도메인을 AWS router 53으로 기관 이전하는 작업을 수행합니다.

## 4. Choose a domain name

간단한 절차로 도메인을 등록 할 수 있습니다.

1. 도메인 검색 -> 2. 연락처 입력  -> 3. 확인 및 결제

(1: Domain Search -> 2: Contact Details -> 3. Verify & Purchase)

### [step 1]

![](/static/images/aws-register-domain/Screenshot_2017-12-27-233545-GMT9-1024x857.png)

### [step 2]

연락처 정보를 입력합니다.

오른쪽 결제 금액이 나오는데 실제 결제되는 금액은 부가세 까지 해서

$13.20가 결제 됩니다.

![](/static/images/aws-register-domain/Screenshot_2017-12-27-234201-GMT9-1024x933.png)

### [step 3]

입력 한 정보가 맞는 지 확인

그리고 [동의: agree 채크박스 선택 ] -> [결제 : Complete Purchase]

하면 도메인 등록 프로세스가 시작되며, 도메인 등록이 완료 됩니다.

![](/static/images/aws-register-domain/Screenshot_2017-12-27-234522-GMT9-1024x935.png)

## 5. 도메인 Name Server 또는 A 레코드 입력하기

[Dashboard] -> [Hosted zones]에서 Name 서버 , 레코드 등록을 할 수 있습니다.

![](/static/images/aws-register-domain/Screenshot_2017-12-27-235123-GMT9-1024x435.png)

2018년에는 kr을 제외한 도메인을 AWS으로 이전 할 계획을 하고 있었고

테스트로 신규 도메인을 등록해서 정상적으로 운영되고 있습니다.

2018-01-09 추가 : 주의!! 매월 도메인 1개당 0.5불씩 과금됩니다.
