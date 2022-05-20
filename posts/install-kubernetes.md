---
title:  "kubernetes(k8s)설치 - kubespray"
subtitle: "kubespray"
tags:
- kubernetes
- k8s
date: 2020-04-05
image: https://user-images.githubusercontent.com/34048253/138473541-ca9eef72-8d1f-4a24-8115-fe96b01f1eb0.png
---


## 개요

> `kubespray`로 `kubernetes(k8s)` 설치하기  
> 최신 업데이트 :2021년 2월 14일

- 목차
  - [서버구성](#서버구성)
  - [쿠버네티스 설치](#쿠버네티스-설치)
  - [트러블슈팅](#트러블슈팅)

`kubespary`를 통해 설치 하면
생각보다 쉽게 쿠버네티스가 설치가 된다.

작년에 실습 삼아 쿠버네티스를 설치 했었다.  

AWS에 매달 5만원 넘게 내는 비용이 부담스러워
홈서버에 쿠버네티스를 운영하기로 했다.
2TB SSD를 추가 장착하고 사용할 node5개에 ubuntu 를  설치했다.
좀더 자원을 주고 싶었으나,

windows server 자원을 제외한 나머지로 구성하였습니다.

`2020.04.01`에 설치한 방법은 `inventory.ini`을 직접 수정 한 버전임.  
[쿠버네티스설치\_이전버전](#쿠버네티스-설치2)

`2021.2.14` 버전은
아래의 주소의 README 문서대로 설치하였습니다.
[https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)

## 서버구성

- Ubuntu18.04 host 5대
  (ubuntu20.04 설치 실패 해서 18.04로 재 설치)
- VMWare ESXI 6.7

  | Name  | CPU   | Memory | SSD   | IP             |
    | ----- | ----- | ------ | ----- | -------------- |
  | node1 | 4core | 12GB   | 350GB | 192.168.55.3   |
  | node2 | 4core | 12GB   | 350GB | 192.168.55.203 |
  | node3 | 4core | 12GB   | 350GB | 192.168.55.223 |
  | node4 | 4core | 12GB   | 350GB | 192.168.55.109 |
  | node5 | 4core | 12GB   | 350GB | 192.168.55.249 |

- 물리서버

  - CPU: 2X Intel Xeon E5-2670 2.6GHz 16-Cores
  - RAM: 96GB
  - SSD: 256GB, 2TB
  - HDD: 4TB

- 설치과정
  <!-- ![](https://sundoforce.github.io/assets/img/code/install_ubuntu16.04.png)   -->
  <!-- ![](https://sundoforce.github.io/assets/img/code/k8s_server_login.png)   -->
  <!-- ![](https://sundoforce.github.io/assets/img/code/esxi-ubutu.png) -->

### net-tools 설치

계정 일반 계정 사용
`sudo apt install net-tools`

### 공개키 키생성

`ssh-keygen -t rsa`

#### 생성 확인

```
la -l .ssh/
total 8
-rw------- 1 kakao kakao 1679 Mar  4 23:08 id_rsa
-rw-r--r-- 1 kakao kakao  395 Mar  4 23:08 id_rsa.pub
```

### 공개키 각 노드에 복사

```
ssh-copy-id 192.168.55.3
ssh-copy-id 192.168.55.203
ssh-copy-id 192.168.55.223
ssh-copy-id 192.168.55.109
ssh-copy-id 192.168.55.249
```

### 접속 테스트

`ssh 192.168.55.203`
로그인 없이 접속되면 성공

### Password 없이 sudo 사용

> 패스워드 입력없이 'sudo' 커멘드 실행

`sudo visudo` 입력

아래의 부분을 찾아서

```
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL) NOPASSWD:ALL

```

`%sudo ALL=(ALL) ALL`를 `%sudo ALL=(ALL) NOPASSWD:ALL`로 수정

### python pip 설치

`sudo apt install python3-pip`

```
sundo@k8s-1:~$ sudo pip3 install --upgrade pip

Collecting pip
Downloading pip-20.3.3-py2.py3-none-any.whl (1.5 MB)
|████████████████████████████████| 1.5 MB 1.3 MB/s
Installing collected packages: pip
Attempting uninstall: pip
Found existing installation: pip 20.0.2
Not uninstalling pip at /usr/lib/python3/dist-packages, outside environment /usr
Can't uninstall 'pip'. No files were found to uninstall.
Successfully installed pip-20.3.3
```

### pip 버전 확인

`user@k8s-1:~$ pip -V`  
`pip 20.3.3 from /usr/local/lib/python3.8/dist-packages/pip (python 3.8)`

### kubespray 복제

`git clone https://github.com/kubernetes-sigs/kubespray.git`

#### 결과

`Cloning into 'kubespray'... remote: Enumerating objects: 49546, done. remote: Total 49546 (delta 0), reused 0 (delta 0), pack-reused 49546 Receiving objects: 100% (49546/49546), 14.66 MiB | 6.20 MiB/s, done. Resolving deltas: 100% (27650/27650), done.`

#### 설치

`sudo pip install -r requirements.txt`

```shell
user@k8s-1:~/kubespray$ sudo pip install -r requirements.txt
Collecting ansible==2.9.6
  Downloading ansible-2.9.6.tar.gz (14.2 MB)
     |████████████████████████████████| 14.2 MB 7.9 MB/s
Requirement already satisfied: PyYAML in /usr/lib/python3/dist-packages (from ansible==2.9.6->-r requirements.txt (line 1)) (5.3.1)
Requirement already satisfied: cryptography in /usr/lib/python3/dist-packages (from ansible==2.9.6->-r requirements.txt (line 1)) (2.8)
Collecting jinja2==2.11.1
  Downloading Jinja2-2.11.1-py2.py3-none-any.whl (126 kB)
     |████████████████████████████████| 126 kB 12.9 MB/s
Requirement already satisfied: MarkupSafe>=0.23 in /usr/lib/python3/dist-packages (from jinja2==2.11.1->-r requirements.txt (line 2)) (1.1.0)
Collecting jmespath==0.9.5
  Downloading jmespath-0.9.5-py2.py3-none-any.whl (24 kB)
Collecting netaddr==0.7.19
  Downloading netaddr-0.7.19-py2.py3-none-any.whl (1.6 MB)
     |████████████████████████████████| 1.6 MB 11.2 MB/s
Collecting pbr==5.4.4
  Downloading pbr-5.4.4-py2.py3-none-any.whl (110 kB)
     |████████████████████████████████| 110 kB 1.7 MB/s
Collecting ruamel.yaml==0.16.10
  Downloading ruamel.yaml-0.16.10-py2.py3-none-any.whl (111 kB)
     |████████████████████████████████| 111 kB 11.9 MB/s
Collecting ruamel.yaml.clib>=0.1.2
  Downloading ruamel.yaml.clib-0.2.2-cp38-cp38-manylinux1_x86_64.whl (578 kB)
     |████████████████████████████████| 578 kB 12.4 MB/s
Building wheels for collected packages: ansible
  Building wheel for ansible (setup.py) ... done
  Created wheel for ansible: filename=ansible-2.9.6-py3-none-any.whl size=16164186 sha256=0e4c703cf1639de65d09967cfffb84368b674f686ccc73963a8c1cfa34aaecef
  Stored in directory: /root/.cache/pip/wheels/b6/4a/7c/2ca8a5588b8df841aabfbc38fffcc8145406cbd092a340a243
Successfully built ansible
Installing collected packages: ruamel.yaml.clib, jinja2, ruamel.yaml, pbr, netaddr, jmespath, ansible
  Attempting uninstall: jinja2
    Found existing installation: Jinja2 2.10.1
    Uninstalling Jinja2-2.10.1:
      Successfully uninstalled Jinja2-2.10.1
Successfully installed ansible-2.9.6 jinja2-2.11.1 jmespath-0.9.5 netaddr-0.7.19 pbr-5.4.4 ruamel.yaml-0.16.10 ruamel.yaml.clib-0.2.2



```

#### 앤써블 버전 확인

```shell
user@k8s-1:~/kubespray$ ansible --version
ansible 2.9.6
  config file = /home/sundo/kubespray/ansible.cfg
  configured module search path = ['/home/sundo/kubespray/library']
  ansible python module location = /usr/local/lib/python3.8/dist-packages/ansible
  executable location = /usr/local/bin/ansible
  python version = 3.8.5 (default, Jul 28 2020, 12:59:40) [GCC 9.3.0]
```

## 쿠버네티스 설치

> 작업 디렉토리 `kubespray`

`cp -rfp inventory/sample inventory/mycluster`

#### 각각 노드 IP 적어줌

`declare -a IPS=(192.168.55.51 192.168.55.76 192.168.55.87 192.168.55.75 192.168.55.228)`

### 설정 파일 생성

`CONFIG_FILE=inventory/mycluster/hosts.yml python3 contrib/inventory_builder/inventory.py ${IPS[@]}`

```shell
~/kubespray$ CONFIG_FILE=inventory/mycluster/hosts.yml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
DEBUG: Adding group all
DEBUG: Adding group kube-master
DEBUG: Adding group kube-node
DEBUG: Adding group etcd
DEBUG: Adding group k8s-cluster
DEBUG: Adding group calico-rr
Usage: inventory.py ip1 [ip2 ...]
Examples: inventory.py 10.10.1.3 10.10.1.4 10.10.1.5

Available commands:
help - Display this message
print_cfg - Write inventory file to stdout
print_ips - Write a space-delimited list of IPs from "all" group
print_hostnames - Write a space-delimited list of Hostnames from "all" group

Advanced usage:
Add another host after initial creation: inventory.py 10.10.1.5
Add range of hosts: inventory.py 10.10.1.3-10.10.1.5
Add hosts with different ip and access ip: inventory.py 10.0.0.1,192.168.10.1 10.0.0.2,192.168.10.2 10.0.0.3,192.168.10.3
Add hosts with a specific hostname, ip, and optional access ip: first,10.0.0.1,192.168.10.1 second,10.0.0.2 last,10.0.0.3
Delete a host: inventory.py -10.10.1.3
Delete a host by id: inventory.py -node1

Configurable env vars:
DEBUG                   Enable debug printing. Default: True
CONFIG_FILE             File to write config to Default: ./inventory/sample/hosts.yaml
HOST_PREFIX             Host prefix for generated hosts. Default: node
SCALE_THRESHOLD         Separate ETCD role if # of nodes >= 50
MASSIVE_SCALE_THRESHOLD Separate K8s master and ETCD if # of nodes >= 200
```

### 설치

아래의 명령어를 입력하면
`ansible-playbook -i inventory/mycluster/hosts.yml --become --become-user=root cluster.yml`

깔끔히 설치 된다.
`Ubuntu20.04버전은 에러 남`

## 이전 작성 글

#### ssh key 생성

```
ssh-keygen -t rsa

Generating public/private rsa key pair.
Enter file in which to save the key (/home/kakao/.ssh/id_rsa):
Created directory '/home/kakao/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/kakao/.ssh/id_rsa.
Your public key has been saved in /home/kakao/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:jf2Hdsglf6RZupEHLOiwL942CFh/B5HzO41GJpfKsu4 kakao@kakao-1
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|                 |
|                 |
|                 |
|                 |
|                 |
+----[SHA256]-----+

```

#### 생성 확인

```
la -l .ssh/
total 8
-rw------- 1 kakao kakao 1679 Mar  4 23:08 id_rsa
-rw-r--r-- 1 kakao kakao  395 Mar  4 23:08 id_rsa.pub
```

#### 내용 확인

```
cat .ssh/id_rsa.pub

```

#### 각 노드로 key 복사

```
$ssh-copy-id kakao@192.168.55.206 # 마스터 노드 #1
$ssh-copy-id kakao@192.168.55.117
$ssh-copy-id kakao@192.168.55.183
$ssh-copy-id kakao@192.168.55.105
$ssh-copy-id kakao@192.168.55.199

# 작업 피시
$ssh-copy-id kakao@192.168.55.131

ssh-copy-id root@192.168.55.117
ssh-copy-id root@192.168.55.183
ssh-copy-id root@192.168.55.105
ssh-copy-id root@192.168.55.199
```

#### 각 노드로 접속해 ssh key 확인

ex

```
kakao@kakao-2:~$ cat .ssh/authorized_keys
```

#### 마스터 노드에서 pip 설치

kakao1번 서버

```
sudo apt -y install python-pip
```

```
pip install --upgrade pip
```

#### kubespray 다운

```
$git clone https://github.com/kubernetes-sigs/kubespray.git
Cloning into 'kubespray'...
remote: Enumerating objects: 13, done.
remote: Counting objects: 100% (13/13), done.
remote: Compressing objects: 100% (13/13), done.
remote: Total 40892 (delta 6), reused 0 (delta 0), pack-reused 40879
Receiving objects: 100% (40892/40892), 11.85 MiB | 4.24 MiB/s, done.
Resolving deltas: 100% (22701/22701), done.
Checking connectivity... done.
```

#### 설치

```
$sudo pip install -r requirements.txt
```

```
WARNING: pip is being invoked by an old script wrapper. This will fail in a future version of pip.
Please see https://github.com/pypa/pip/issues/5599 for advice on fixing the underlying issue.
To avoid this problem you can invoke Python with '-m pip' instead of running pip directly.
DEPRECATION: Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. A future version of pip will drop support for Python 2.7. More details about Python 2 support in pip, can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support
WARNING: The directory '/home/kakao/.cache/pip' or its parent directory is not owned or is not writable by the current user. The cache has been disabled. Check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLEOFError(8, u'EOF occurred in violation of protocol (_ssl.c:590)'),)': /simple/ansible/
Collecting ansible==2.7.12
  Downloading ansible-2.7.12.tar.gz (11.9 MB)
     |████████████████████████████████| 11.9 MB 4.8 MB/s
Collecting jinja2==2.10.1
  Downloading Jinja2-2.10.1-py2.py3-none-any.whl (124 kB)
     |████████████████████████████████| 124 kB 192 kB/s
Collecting netaddr==0.7.19
  Downloading netaddr-0.7.19-py2.py3-none-any.whl (1.6 MB)
     |████████████████████████████████| 1.6 MB 756 kB/s
Collecting pbr==5.2.0
  Downloading pbr-5.2.0-py2.py3-none-any.whl (107 kB)
     |████████████████████████████████| 107 kB 189 kB/s
Collecting hvac==0.8.2
  Downloading hvac-0.8.2-py2.py3-none-any.whl (91 kB)
     |████████████████████████████████| 91 kB 132 kB/s
Collecting jmespath==0.9.4
  Downloading jmespath-0.9.4-py2.py3-none-any.whl (24 kB)
Collecting ruamel.yaml==0.15.96
  Downloading ruamel.yaml-0.15.96-cp27-cp27mu-manylinux1_x86_64.whl (601 kB)
     |████████████████████████████████| 601 kB 380 kB/s
Collecting PyYAML
  Downloading PyYAML-5.3.tar.gz (268 kB)
     |████████████████████████████████| 268 kB 747 kB/s
Collecting paramiko
  Downloading paramiko-2.7.1-py2.py3-none-any.whl (206 kB)
     |████████████████████████████████| 206 kB 712 kB/s
Collecting cryptography
  Downloading cryptography-2.8-cp27-cp27mu-manylinux2010_x86_64.whl (2.3 MB)
     |████████████████████████████████| 2.3 MB 337 kB/s
Requirement already satisfied: setuptools in /usr/lib/python2.7/dist-packages (from ansible==2.7.12->-r requirements.txt (line 1)) (20.7.0)
Collecting MarkupSafe>=0.23
  Downloading MarkupSafe-1.1.1-cp27-cp27mu-manylinux1_x86_64.whl (24 kB)
Collecting requests>=2.21.0
  Downloading requests-2.23.0-py2.py3-none-any.whl (58 kB)
     |████████████████████████████████| 58 kB 23.1 MB/s
Collecting ruamel.ordereddict; platform_python_implementation == "CPython" and python_version <= "2.7"
  Downloading ruamel.ordereddict-0.4.14-cp27-cp27mu-manylinux1_x86_64.whl (93 kB)
     |████████████████████████████████| 93 kB 380 kB/s
Collecting pynacl>=1.0.1
  Downloading PyNaCl-1.3.0-cp27-cp27mu-manylinux1_x86_64.whl (762 kB)
     |████████████████████████████████| 762 kB 336 kB/s
Collecting bcrypt>=3.1.3
  Downloading bcrypt-3.1.7-cp27-cp27mu-manylinux1_x86_64.whl (59 kB)
     |████████████████████████████████| 59 kB 23.3 MB/s
Collecting six>=1.4.1
  Downloading six-1.14.0-py2.py3-none-any.whl (10 kB)
Collecting cffi!=1.11.3,>=1.8
  Downloading cffi-1.14.0-cp27-cp27mu-manylinux1_x86_64.whl (387 kB)
     |████████████████████████████████| 387 kB 315 kB/s
Collecting ipaddress; python_version < "3"
  Downloading ipaddress-1.0.23-py2.py3-none-any.whl (18 kB)
Collecting enum34; python_version < "3"
  Downloading enum34-1.1.9-py2-none-any.whl (11 kB)
Collecting certifi>=2017.4.17
  Downloading certifi-2019.11.28-py2.py3-none-any.whl (156 kB)
     |████████████████████████████████| 156 kB 549 kB/s
Collecting idna<3,>=2.5
  Downloading idna-2.9-py2.py3-none-any.whl (58 kB)
     |████████████████████████████████| 58 kB 22.1 MB/s
Collecting chardet<4,>=3.0.2
  Downloading chardet-3.0.4-py2.py3-none-any.whl (133 kB)
     |████████████████████████████████| 133 kB 647 kB/s
Collecting urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1
  Downloading urllib3-1.25.8-py2.py3-none-any.whl (125 kB)
     |████████████████████████████████| 125 kB 682 kB/s
Collecting pycparser
  Downloading pycparser-2.20-py2.py3-none-any.whl (112 kB)
     |████████████████████████████████| 112 kB 516 kB/s
Building wheels for collected packages: ansible, PyYAML
  Building wheel for ansible (setup.py) ... done
  Created wheel for ansible: filename=ansible-2.7.12-py2-none-any.whl size=9430239 sha256=dae0a74b4e52c8a75f6f4d32ebca19920335a8da22fdb8a27129c8ce10763307
  Stored in directory: /tmp/pip-ephem-wheel-cache-IRMPOh/wheels/35/16/79/a507de0ed355e1b8030e0bf6f19b469dfc3600cd0ce4567406
  Building wheel for PyYAML (setup.py) ... done
  Created wheel for PyYAML: filename=PyYAML-5.3-cp27-cp27mu-linux_x86_64.whl size=46466 sha256=eca64dfe0ad2946927bbe688fea1b1bbefe6aad4c85867527dae9d6871ca46cb
  Stored in directory: /tmp/pip-ephem-wheel-cache-IRMPOh/wheels/06/0f/01/cdac9a236377daed23bce2de82d8cc036abc5717f63bd8268c
Successfully built ansible PyYAML
Installing collected packages: MarkupSafe, jinja2, PyYAML, six, pycparser, cffi, ipaddress, enum34, cryptography, pynacl, bcrypt, paramiko, ansible, netaddr, pbr, certifi, idna, chardet, urllib3, requests, hvac, jmespath, ruamel.ordereddict, ruamel.yaml
Successfully installed MarkupSafe-1.1.1 PyYAML-5.3 ansible-2.7.12 bcrypt-3.1.7 certifi-2019.11.28 cffi-1.14.0 chardet-3.0.4 cryptography-2.8 enum34-1.1.9 hvac-0.8.2 idna-2.9 ipaddress-1.0.23 jinja2-2.10.1 jmespath-0.9.4 netaddr-0.7.19 paramiko-2.7.1 pbr-5.2.0 pycparser-2.20 pynacl-1.3.0 requests-2.23.0 ruamel.ordereddict-0.4.14 ruamel.yaml-0.15.96 six-1.14.0 urllib3-1.25.8
```

#### inventory.ini 파일 수정

```
vi inventory/mycluster/inventory.ini

```

inventory.ini 파일 내용  
host명으로 수정해 준다.

```
## Configure 'ip' variable to bind kubernetes services on a
# ## different ip than the default iface
# ## We should set etcd_member_name for etcd cluster. The node that is not a etcd member do not need to set the value, or can set the empty string value.
[all]
kakao-1 ansible_host=192.168.55.206 # ip=192.168.55.206 etcd_member_name=etcd1
kakao-2 ansible_host=192.168.55.117 # ip=192.168.55.117 etcd_member_name=etcd1
kakao-3 ansible_host=192.168.55.183 # ip=192.168.55.183 etcd_member_name=etcd1
kakao-4 ansible_host=192.168.55.105 # ip=192.168.55.105 etcd_member_name=etcd1
kakao-5 ansible_host=192.168.55.199 # ip=192.168.55.199 etcd_member_name=etcd1


# ## configure a bastion host if your nodes are not directly reachable
# bastion ansible_host=x.x.x.x ansible_user=some_user

[kube-master]
# node1
# node2
kakao-1
kakao-2
kakao-3

[etcd]
kakao-1
kakao-2
kakao-3


[kube-node]
kakao-4
kakao-5

[calico-rr]

[k8s-cluster:children]
kube-master
kube-node
calico-rr
```

#### 설치 확인

노드 가져오기

```
root@kakao-1:~# kubectl get node
NAME      STATUS   ROLES    AGE     VERSION
kakao-1   Ready    master   5m52s   v1.16.6
kakao-2   Ready    master   5m1s    v1.16.6
kakao-3   Ready    master   5m1s    v1.16.6
kakao-4   Ready    <none>   3m18s   v1.16.6
kakao-5   Ready    <none>   3m18s   v1.16.6
```

## 트러블슈팅

ubuntu 20.04는 설치가 안되었던 반면

```shell
SK [download : download_container | Download image if required] *********************************************************************
fatal: [node1 -> 192.168.55.51]: FAILED! => {"attempts": 4, "changed": true, "cmd": ["/usr/bin/docker", "pull", "quay.io/coreos/etcd@sha256:None"], "delta": "0:00:00.064617", "end": "2021-01-08 13:49:02.595504", "msg": "non-zero return code", "rc": 1, "start": "2021-01-08 13:49:02.530887", "stderr": "invalid reference format", "stderr_lines": ["invalid reference format"], "stdout": "", "stdout_lines": []}
FAILED - RETRYING: download_container | Download image if required (2 retries left).
FAILED - RETRYING: download_container | Download image if required (2 retries left).
FAILED - RETRYING: download_container | Download image if required (1 retries left).
FAILED - RETRYING: download_container | Download image if required (1 retries left).
fatal: [node2 -> 192.168.55.76]: FAILED! => {"attempts": 4, "changed": true, "cmd": ["/usr/bin/docker", "pull", "quay.io/coreos/etcd@sha256:None"], "delta": "0:00:00.052715", "end": "2021-01-08 13:49:18.261204", "msg": "non-zero return code", "rc": 1, "start": "2021-01-08 13:49:18.208489", "stderr": "invalid reference format", "stderr_lines": ["invalid reference format"], "stdout": "", "stdout_lines": []}
fatal: [node3 -> 192.168.55.87]: FAILED! => {"attempts": 4, "changed": true, "cmd": ["/usr/bin/docker", "pull", "quay.io/coreos/etcd@sha256:None"], "delta": "0:00:00.053943", "end": "2021-01-08 13:49:18.326764", "msg": "non-zero return code", "rc": 1, "start": "2021-01-08 13:49:18.272821", "stderr": "invalid reference format", "stderr_lines": ["invalid reference format"], "stdout": "", "stdout_lines": []}

```

버전을 18.04로 변경하니 해결 됨.

16.04에는 2020-04에 `inventory.ini` 파일을 직접수정 사용

## 추가로 설치 한것 들

- grafana
- dashboard
- Kibana
