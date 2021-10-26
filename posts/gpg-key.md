---
layout: post
title:  "github verified 커밋"
subtitle: "잔디밭이 안심어 져요"
author: "sdk(muse kim)"
catalog: true
header-mask:  0.3
tags:
-  gnuboard
-  kubernetes
-  k8s
date:   2020-04-21
multilingual: flase
---


- 목차
   - [개요](#개요)
   - [GPG 인증 작동](#GPG-인증-작동)
   - [GPG 키 얻기](#GPG-키-얻기)
   - [gpg 설치](#gpg-설치)
      - [새로운 GPG 키 생성](#새로운-GPG-키 생성)
      - [GPG 키로 커밋에 서명](#GPG-키로-커밋에-서명)
      - [응용](#응용)
   - [키 목록](#키-목록)
      - [추가로 알게 된 사항](#추가로-알게-된-사항
        )

## 개요
![](https://sundoforce.github.io/assets/img/code/git.png)
열심히 commit했는데
커밋한게 github에 잔디가 심어지지 않았다.

구굴링 해보면 commit한 이메일이 같아야 된다고 했는데  
내가 커밋한 코드에는 인증 뱃지가 없다...

```
git config  --list | grep user                                          ✔
user.email=admin@sundo.kim
```

![](https://sundoforce.github.io/assets/img/code/Verified.png)


깃 허브 잔디는 Verified 표시가 있어야 한다고 한다.

그래서 gpg key를 등록 해보았다.

참조: http://blog.brew.com.hk/verified-commits-github/


## GPG 인증 작동

* 개인 및 공개 키로 구성된 GPG 키 쌍을 생성합
* 개인 키를 사용하여 커밋에 서명하고 공개 키를 GitHub와 연결
* GitHub가 GPG 키를 사용하여 서명 된 커밋을 볼 때 제공 한 공개 키를 사용하여 데이터를 해독합니다
* 데이터가 의미가 있으면 GitHub가 커미터를 확인합니다. 아무도 GPG 개인 키를 알지 못하므로 커밋에 서명 할 수 없습니다.

## GPG 키 얻기

기존에 키가 있는지 확인
```
gpg --list-secret-keys --keyid-format LONG
```

```
$gpg --list-secret-keys --keyid-format LONG
gpg: directory `/home/ec2-user/.gnupg' created
gpg: new configuration file `/home/ec2-user/.gnupg/gpg.conf' created
gpg: WARNING: options in `/home/ec2-user/.gnupg/gpg.conf' are not yet active during this run
gpg: keyring `/home/ec2-user/.gnupg/secring.gpg' created
gpg: keyring `/home/ec2-user/.gnupg/pubring.gpg' created
gpg: /home/ec2-user/.gnupg/trustdb.gpg: trustdb created
```

![](https://sundoforce.github.io/assets/img/code/check.png)

## gpg 설치
macOS
```
$brew install gpg
```
amazon ec2 설치
(Amazon linux)
```
sudo yum install gnupg2
yum install gnupg
```

## 새로운 GPG 키 생성

아래의 명령어 입력
```
gpg --full-gen-key
``` 
위 명령어 안될경우
```
gpg --gen-key
```

키 크기가 4096인 RSA 알고리즘 사용   
키가 길수록 공격에 취약하지 않다고 함.




```
 gpg --full-gen-key                                                                                             ✔
gpg (GnuPG) 2.2.20; Copyright (C) 2020 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
  (14) Existing key from card
Your selection? 1
```

1을 입력

```
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
```
4096입력

```
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
```
0입력

````
Key does not expire at all
Is this correct? (y/N) y
````
y입력

```
GnuPG needs to construct a user ID to identify your key.

Real name: sundo kim
Email address: admin@sundo.kim
Comment:
You selected this USER-ID:
    "sundo kim <admin@sundo.kim>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
```
O 입력
```
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key 735A2804E13A821D marked as ultimately trusted
gpg: directory '/Users/sdk/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/Users/sdk/.gnupg/openpgp-revocs.d/9248B38CB7CD3BA93300FECF735A2804E13A821D.rev'
public and secret key created and signed.

pub   rsa4096 2020-04-21 [SC]
      9248B38CB7CD3BA93300FECF735A2804E13A821D
uid                      SDK(SunDoKim) <admin@sundo.kim>
sub   rsa4096 2020-04-21 [E]

```
![](https://sundoforce.github.io/assets/img/code/passphres.png)

```
gpg --armor --export 9248B38CB7CD3BA93300FECF735A2804E13A821D                                              127 ↵
-----BEGIN PGP PUBLIC KEY BLOCK-----


```
키 복사

https://github.com/settings/keys
![](../assets/img/code/gpg_key.png)

여기에서 New GPG Key 입력


## GPG 키로 커밋에 서명


커밋할때 --gpg-sign을 하면되고
git commit --gpg-sign 9248B38CB7CD3BA93300FECF735A2804E13A821D

커밋할때마다 자동으로 할 수 있도록 config에 설정해준다.
```
git config --global user.signingkey 9248B38CB7CD3BA93300FECF735A2804E13A821D  
git config --global commit.gpgsign true
```

![](https://sundoforce.github.io/assets/img/blog/verified_success.png)

## 아마존 리눅스 gpg
amazaon EC2
https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ec2rl_verify.html

amazon ec2 설치
(Amazon linux)
```
sudo yum install gnupg2
yum install gnupg
```

## 응용
키삭제
```
gpg --delete-secret-keys {keyId}
```

```
gpg --delete-secret-keys 6792639B53FF607C
gpg (GnuPG) 2.0.28; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


sec  4096R/53FF607C 2020-04-21 SDK(SunDoKim) <admin@sundo.kim>

Delete this key from the keyring? (y/N) y
This is a secret key! - really delete? (y/N) y
[newcomposer@ip-172-26-12-135 www]$ gpg --delete-secret-keys 01AE3831FCF2817E
gpg (GnuPG) 2.0.28; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


sec  4096R/FCF2817E 2020-04-21 SDK(Sun-Do,Kim) <admin@sundo.kim>

Delete this key from the keyring? (y/N) y
This is a secret key! - really delete? (y/N) y
```

## 키 목록
```
 gpg --list-keys
 ```

```
gpg --list-keys --keyid-format LONG
```

비밀키
```
gpg --list-secret-keys
```

키 비번 변경
```
gpg --edit-key [키 ID]
```

## 추가로 알게 된 사항
출처 : https://okky.kr/article/438013
1. 다른 사람의 것을 포크해 왔는데, 거기다가 커밋을 한거는 저기에 안찍힙니다.

2. private 저장소에 커밋한 경우는 별도로 설정? 을 해주면 저기에 찍힙니다.

3. 마지막에 질문하신 스샷같은 경우, 다음 플러그인을 받아서 설치하시면 이용하실 수 있습니다.

https://chrome.google.com/webstore/detail/isometric-contributions/mjoedlfflcchnleknnceiplgaeoegien

![](https://sundoforce.github.io/assets/img/blog/Contributions.jpg)

커밋 개수가 저 그래프에 찍히는 기준은 다음 링크를 참고하시면 되겠습니다.

https://help.github.com/articles/why-are-my-contributions-not-showing-up-on-my-profile/ 