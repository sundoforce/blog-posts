---
title:  "나의 첫 WASM 기록"
subtitle: "hello wasm"
tags:
-  wasm
date: 2021-05-28
image: https://user-images.githubusercontent.com/34048253/138473541-ca9eef72-8d1f-4a24-8115-fe96b01f1eb0.png
---

# 나의 첫 WASM 기록

> 참고: 웹어샘블리 인 액션 (한빛출판사) 를 읽고 정리한 내용입니다.

## C 코드 작성

c file : 소수인지 판별  
**`calculate_primes.c`**

```c
#include <stdlib.h>
#include <stdio.h>
#include <emscripten.h>

int IsPrime(int value) {
    if (value == 2) { return 1;}
    if (value <=1 || value % 2 == 0) { return 0; }

    for (int i = 3; (i * i) <= value; i += 2) {
        if (value % i == 0) { return 0; }
    }

  return 1
}

int main() {
    int start = 3;
    int end = 100000;

    printf("Prime numbers between %d and %d:\n", start, end);

    for (int i = start; i <= end; i += 2) {
      if (IsPrime(i)) {
        printf("%d", i)
      }
    }
   print("\n")

   return 0;
}

```

## emcc 설치

```
$ git clone https://github.com/emscripten-core/emsdk.git
$ cd emsdk
$ ./emsdk install latest
```

```
Installing SDK 'sdk-releases-upstream-77b065ace39e6ab21446e13f92897f956c80476a-64bit'..
Installing tool 'node-14.15.5-64bit'..
Error: Downloading URL 'https://storage.googleapis.com/webassembly/emscripten-releases-builds/deps/node-v14.15.5-darwin-x64.tar.gz': <urlopen error [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1056)>
Warning: Possibly SSL/TLS issue. Update or install Python SSL root certificates (2048-bit or greater) supplied in Python folder or https://pypi.org/project/certifi/ and try again.
Installation failed!
```

#### 오류날 경우

emsdk파일 상단에  
`ssl import` 추가

**수정파일**: `vi emsdk.py`

```
import ssl
ssl._create_default_https_context = ssl._create_unverified_context
```

**log**

```
./emsdk install latest              ✔
Installing SDK 'sdk-releases-upstream-77b065ace39e6ab21446e13f92897f956c80476a-64bit'..
Installing tool 'node-14.15.5-64bit'..
Downloading: /Users/sdk/IdeaProjects/emsdk/zips/node-v14.15.5-darwin-x64.tar.gz from https://storage.googleapis.com/webassembly/emscripten-releases-builds/deps/node-v14.15.5-darwin-x64.tar.gz, 31560456 Bytes
Unpacking '/Users/sdk/IdeaProjects/emsdk/zips/node-v14.15.5-darwin-x64.tar.gz' to '/Users/sdk/IdeaProjects/emsdk/node/14.15.5_64bit'
Done installing tool 'node-14.15.5-64bit'.
Installing tool 'python-3.9.2-1-64bit'..
Downloading: /Users/sdk/IdeaProjects/emsdk/zips/python-3.9.2-1-macos-x86_64.tar.gz from https://storage.googleapis.com/webassembly/emscripten-releases-builds/deps/python-3.9.2-1-macos-x86_64.tar.gz, 32022000 Bytes
Unpacking '/Users/sdk/IdeaProjects/emsdk/zips/python-3.9.2-1-macos-x86_64.tar.gz' to '/Users/sdk/IdeaProjects/emsdk/python/3.9.2-1_64bit'
Done installing tool 'python-3.9.2-1-64bit'.
Installing tool 'releases-upstream-77b065ace39e6ab21446e13f92897f956c80476a-64bit'..
Downloading: /Users/sdk/IdeaProjects/emsdk/zips/77b065ace39e6ab21446e13f92897f956c80476a-wasm-binaries.tbz2 from https://storage.googleapis.com/webassembly/emscripten-releases-builds/mac/77b065ace39e6ab21446e13f92897f956c80476a/wasm-binaries.tbz2, 165357215 Bytes
Unpacking '/Users/sdk/IdeaProjects/emsdk/zips/77b065ace39e6ab21446e13f92897f956c80476a-wasm-binaries.tbz2' to '/Users/sdk/IdeaProjects/emsdk/upstream'
Done installing tool 'releases-upstream-77b065ace39e6ab21446e13f92897f956c80476a-64bit'.
Done installing SDK 'sdk-releases-upstream-77b065ace39e6ab21446e13f92897f956c80476a-64bit'.
```

### activate lates

`./emsdk activate latest`

```
$ . ./emsdk activate latest            ✔
Setting the following tools as active:
   node-14.15.5-64bit
   python-3.9.2-1-64bit
   releases-upstream-77b065ace39e6ab21446e13f92897f956c80476a-64bit

Next steps:
- To conveniently access emsdk tools from the command line,
  consider adding the following directories to your PATH:
    /Users/sdk/IdeaProjects/emsdk
    /Users/sdk/IdeaProjects/emsdk/node/14.15.5_64bit/bin
    /Users/sdk/IdeaProjects/emsdk/upstream/emscripten
- This can be done for the current shell by running:
    source "/Users/sdk/IdeaProjects/emsdk/emsdk_env.sh"
- Configure emsdk in your bash profile by running:
    echo 'source "/Users/sdk/IdeaProjects/emsdk/emsdk_env.sh"' >> $HOME/.bash_profile
```

### 퍼미션 변경후 env.sh 실행

**sh 파일 퍼미션 변경**  
`$chmod a+x emsdk_env.sh`  
**실행**  
`$ ./emsdk_env.sh`

```
./emsdk_env.sh                      ✔

Adding directories to PATH:
PATH += /Users/sdk/IdeaProjects/emsdk/upstream/emscripten
PATH += /Users/sdk/IdeaProjects/emsdk/node/14.15.5_64bit/bin

Setting environment variables:
PATH = /Users/sdk/IdeaProjects/emsdk/upstream/emscripten:/Users/sdk/IdeaProjects/emsdk/node/14.15.5_64bit/bin:/Users/sdk/IdeaProjects/emsdk:/Library/Frameworks/Python.framework/Versions/2.7/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/go/bin:/usr/local/share/dotnet:~/.dotnet/tools:/Library/Apple/usr/bin:/Library/Frameworks/Mono.framework/Versions/Current/Commands
EMSDK_NODE = /Users/sdk/IdeaProjects/emsdk/node/14.15.5_64bit/bin/node
EMSDK_PYTHON = /Users/sdk/IdeaProjects/emsdk/python/3.9.2-1_64bit/bin/python3
SSL_CERT_FILE = /Users/sdk/IdeaProjects/emsdk/python/3.9.2-1_64bit/lib/python3.9/site-packages/certifi/cacert.pem
```

#### .profile 적용

`$ source ./emsdk_env.sh`

```
source ./emsdk_env.sh              ✔
Adding directories to PATH:
PATH += /Users/sdk/IdeaProjects/emsdk/upstream/emscripten
PATH += /Users/sdk/IdeaProjects/emsdk/node/14.15.5_64bit/bin

Setting environment variables:
PATH = /Users/sdk/IdeaProjects/emsdk/upstream/emscripten:/Users/sdk/IdeaProjects/emsdk/node/14.15.5_64bit/bin:/Users/sdk/IdeaProjects/emsdk:/Library/Frameworks/Python.framework/Versions/2.7/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/go/bin:/usr/local/share/dotnet:~/.dotnet/tools:/Library/Apple/usr/bin:/Library/Frameworks/Mono.framework/Versions/Current/Commands
EMSDK_NODE = /Users/sdk/IdeaProjects/emsdk/node/14.15.5_64bit/bin/node
EMSDK_PYTHON = /Users/sdk/IdeaProjects/emsdk/python/3.9.2-1_64bit/bin/python3
SSL_CERT_FILE = /Users/sdk/IdeaProjects/emsdk/python/3.9.2-1_64bit/lib/python3.9/site-packages/certifi/cacert.pem
```

### Web Assembly paths

`source "/Users/sdk/IdeaProjects/emsdk/emsdk_env.sh" >> $HOME/.bash_profile`  
**stdout**

```
source  "/Users/sdk/IdeaProjects/emsdk/emsdk_env.sh" >> $HOME/.bash_profile                                                    SIGINT(2) ↵
Adding directories to PATH:
PATH += /Users/sdk/IdeaProjects/emsdk
PATH += /Users/sdk/IdeaProjects/emsdk/upstream/emscripten
PATH += /Users/sdk/IdeaProjects/emsdk/node/14.15.5_64bit/bin

Setting environment variables:
PATH = /Users/sdk/IdeaProjects/emsdk:/Users/sdk/IdeaProjects/emsdk/upstream/emscripten:/Users/sdk/IdeaProjects/emsdk/node/14.15.5_64bit/bin:/Library/Frameworks/Python.framework/Versions/2.7/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/go/bin:/usr/local/share/dotnet:~/.dotnet/tools:/Library/Apple/usr/bin:/Library/Frameworks/Mono.framework/Versions/Current/Commands
EMSDK = /Users/sdk/IdeaProjects/emsdk
EM_CONFIG = /Users/sdk/IdeaProjects/emsdk/.emscripten
EMSDK_NODE = /Users/sdk/IdeaProjects/emsdk/node/14.15.5_64bit/bin/node
EMSDK_PYTHON = /Users/sdk/IdeaProjects/emsdk/python/3.9.2-1_64bit/bin/python3
```

### 설치 확인

`emcc -v`

```
emcc -v                             ✔
emcc (Emscripten gcc/clang-like replacement + linker emulating GNU ld) 2.0.23 (b15ca40ca02f469de3bf9a4de67a2655aede8974)
clang version 13.0.0 (/opt/s/w/ir/cache/git/chromium.googlesource.com-external-github.com-llvm-llvm--project 5852582532b3eb3ea8da51a1e272d8d017bd36c9)
Target: wasm32-unknown-emscripten
Thread model: posix
InstalledDir: /Users/sdk/IdeaProjects/emsdk/upstream/bin

source ./emsdk_env.sh


ource ./emsdk_env.sh               ✔
Adding directories to PATH:
PATH += /Users/sdk/IdeaProjects/emsdk
PATH += /Users/sdk/IdeaProjects/emsdk/upstream/emscripten
PATH += /Users/sdk/IdeaProjects/emsdk/node/14.15.5_64bit/bin

Setting environment variables:
PATH = /Users/sdk/IdeaProjects/emsdk:/Users/sdk/IdeaProjects/emsdk/upstream/emscripten:/Users/sdk/IdeaProjects/emsdk/node/14.15.5_64bit/bin:/Library/Frameworks/Python.framework/Versions/2.7/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/go/bin:/usr/local/share/dotnet:~/.dotnet/tools:/Library/Apple/usr/bin:/Library/Frameworks/Mono.framework/Versions/Current/Commands
EMSDK = /Users/sdk/IdeaProjects/emsdk
EM_CONFIG = /Users/sdk/IdeaProjects/emsdk/.emscripten
EMSDK_NODE = /Users/sdk/IdeaProjects/emsdk/node/14.15.5_64bit/bin/node
EMSDK_PYTHON = /Users/sdk/IdeaProjects/emsdk/python/3.9.2-1_64bit/bin/python3
SSL_CERT_FILE = /Users/sdk/IdeaPr

```

$ `emcc --version`

```
emcc --version                    1 ↵
emcc (Emscripten gcc/clang-like replacement + linker emulating GNU ld) 2.0.23 (b15ca40ca02f469de3bf9a4de67a2655aede8974)
Copyright (C) 2014 the Emscripten authors (see AUTHORS.txt)
This is free and open source software under the MIT license.
There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

```

### WASM파일 생성

C 코드를 엠스크립튼을 이용해 웹어셈블리 파일로 컴파일  
$ `emcc calculate_primes.c -o html_template.html`
생성된 파일

- `.wasm` : 어샘블리모듈
- `.js`: 자바 스크립트
- `.html`: html 파일

#### emcc arugments

`-o `: 플래그 값  
`html_template.html`: html까지 입력시 - 전체 자동생성(wasm, js. html)

### 확인

```
ls -al                    ✔
total 760
drwxr-xr-x  8 sdk  staff     256 May 28 08:12 .
drwxr-xr-x  5 sdk  staff     160 May 28 07:37 ..
-rw-r--r--  1 sdk  staff     548 May 28 08:12 calculate_primes.c
-rw-r--r--  1 sdk  staff  121901 May 28 08:12 html
-rwxr-xr-x  1 sdk  staff   12354 May 28 08:12 html.wasm
-rw-r--r--  1 sdk  staff  102683 May 28 08:12 html_template.html
-rw-r--r--  1 sdk  staff  121910 May 28 08:12 html_template.js
-rwxr-xr-x  1 sdk  staff   12354 May 28 08:12 html_template.wasm
```

### 서버 실행

## python2

`python -m SimpleHTTPServer 8080`

### python3

`python3 -m http.server 8080`

or  
`emrun --port 8080 test.html`

```
python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

#### 접속

![2021-05-28-wasm01](https://user-images.githubusercontent.com/16316626/138916061-14c710b3-faff-45b5-8702-50d27edd8e84.png)


#### https//localhost:8080/html_template.html

![2021-05-28-wasm02](https://user-images.githubusercontent.com/16316626/138916093-4355a666-83e7-4fd4-8ab1-f0cbd0303c08.png)

## js 만 생성

**html**  
안에 `<script src="js_plumbing.js"></script>` 작성

```
1 <!DOCTYPE html>
2 <html>
3   <head>
4     <meta charset="utf-8">
5   </head>
6   <body>
7       HTML page I created for my WebAssembly module.<br/>
8       WebAssembly 모듈 용으로 만든 HTML 페이지입니다.
9   <script src="js_plumbing.js"></script>
10   </body>
11 </html>
```

#### js파일 생성

`$emcc calculate_primes.c -o js_plumbing.js`

실행
![03](https://user-images.githubusercontent.com/16316626/138916164-f777a827-5407-4be9-86e1-dd172c40ea38.png)
## wasm
파일만 생성

> 엠스크립튼으로 웹어셈블리 파일만 생성  
> C/C++ 코드를 웹 어셈블리로 컴파일  
> 다른 파일은 생성하지 않음  
> side module(복수 모듈)을 만들 때 사용함.

#### side module이 필요한 경우

- 메인 모듈을 컴파일 하고 C 라이브러리 함수를 메인 모듈에 둠
- C 라이브러리가 전혀 필요 없는 경우
- 웹어셈블러 다른 컴파일이 있다면 수동으로 로드 할수 있음

**side_module.c**

```
int Increment(int value) {
  return (value + 1);
}

int Decrement(int value) {
  return (value - 1);
}

```

### 컴파일

`emcc side_module.c -s SIDE_MODULE=2 -O1 -s EXPORTED_FUNCTIONS=['_Increment'] -o side_module.wasm`

#### 설명: emcc에 플래그 지정

`-s SIDE_MODULE=2`  
`-O1`: 최적화 플래그

> 지정해 주지 않으면 `-O0`이 적용되어 모듈 로드 시 링크 에러발생

`-s EXPORTED_FUNCTIONS`: 변수명 앞에 언더스코어(`_`)추가해야 함

> 변수 여러개 사용  
> `-s "EXPORTED_FUNCTIONS=['_Increment', '_Decrement']"`

```
emcc side_module.c
   -s SIDE_MODULE=2 -O1
   -s "EXPORTED_FUNCTIONS=['_Increment', '_Decrement']"
   -o side_module.wasm
```

#### 트러블 슈팅

```
zsh: no matches found: EXPORTED_FUNCTIONS=[_Incrrement]
```

EXPORTED_FUNCTION을 `""`로 감싸 줌

```
emcc side_moduel.c -s SIDE_MODULE=2 -O1 -s \
"EXPORTED_FUNCTIONS=['_Incrrement']" -o side_moudle.wasm
```

#### 컴파일 된 파일 확인

`$ls -al`

```
 ls -al                   ✔
total 16
drwxr-xr-x   4 sdk  staff  128 May 29 09:35 .
drwxr-xr-x  10 sdk  staff  320 May 29 09:21 ..
-rw-r--r--   1 sdk  staff   54 May 29 09:35 side_moduel.c
-rwxr-xr-x   1 sdk  staff  176 May 29 09:35 side_moudle.wasm
```

### js 파일 , html파일 작성

![04](https://user-images.githubusercontent.com/16316626/138916220-9318c319-634b-49eb-a26b-7f6f274a0ef9.png)
![05](https://user-images.githubusercontent.com/16316626/138916234-fefb6151-e448-4190-84b6-d46f92dfb296.png)


> 트러블 슈팅
> https://github.com/rustwasm/rust-parcel-template/issues/52
> https 에서만 작동 하는 것 같음.
> 한 시간 날림

```
side_module.html:1 Uncaught (in promise) TypeError: Failed to execute 'compile' on 'WebAssembly': Incorrect response MIME type. Expected 'application/wasm'.
```

#### 결과 스킵

c에서 작성한 Increment()를 wasm로 컴파일 후 js 에서 fetch로 받아 함수 실행
기대했던 결과는 console에 출력 되는 것이였음.  
아쉽

```
emcc validate.cpp -o validate.js -s "EXTRA_EXPORTED_RUNTIME_METHODS=['ccall', 'UTF8ToString']"
```