---
title: "도커로 SVN 서버 구축하기"
date: 2019-08-02
tags:
  - docker
  - subversion server
  - SVN server
  - setup svn server
  - garethflowers/svn-server
---

개인적으로 Git을 좋아하지만, 회사에서 SVN을 써서 친해져야합니다.

그래서 SVN서버를 Docker로 설치해보려고 합니다.

## 1. SVN 컨테이너 실행
##### *<U>윈도우는 커맨드 줄 끝의 \ 를 ` 로(esc아래 키) 바꾸시면 됩니다.</U>*

```console
# docker run --name svn-server \
           --detach \
           --volume /my/docker/volume/my-svn-repo:/var/opt/svn \
           --publish 3690:3690 \
           garethflowers/svn-serve

# docker ps
CONTAINER ID        IMAGE                      COMMAND                  CREATED             STATUS                           PORTS                    NAMES
a95a445b29c8        garethflowers/svn-server   "/usr/bin/svnserve -…"   3 seconds ago       Up 1 second (health: starting)   0.0.0.0:3690->3690/tcp   svn-server
```
SVN 서버가 완성되었습니다. 짝짝짝

아래부터는 SVN 설정하는 파트입니다. 넘어가셔도 좋습니다.

## 2. SVN 저장소 만들기

```console
# docker exec -it svn-server svnadmin create test-repo

# docker exec -it svn-server svnadmin info test-repo
Path: test-repo
UUID: 16f1dd00-4a5d-44f2-adcb-24c03d1b441e
Repository Format: 5
Compatible With Version: 1.9.0
Repository Capability: mergeinfo
Filesystem Type: fsfs
Filesystem Format: 7
FSFS Sharded: yes
FSFS Shard Size: 1000
FSFS Shards Packed: 0/0
FSFS Logical Addressing: yes
Configuration File: test-repo/db/fsfs.conf
``` 

## 3. SVN 사용자 추가하기

```console
# docker exec -it svn-server sh

# vi /var/opt/svn/test-repo/conf/passwd
### This file is an example password file for svnserve.
...
[users]
# harry = harryssecret
# sally = sallyssecret
seunghyun = 1234567
```
* 주의 : vi에서 빠져나올때 `'shift+q'`가 동작하지 않습니다. `':' + 'wq' + enter` 키로 빠져나오세요.

## 4. SVN 사용자별 그룹 지정 및 권한 설정

```console
# vi /var/opt/svn/test-repo/conf/authz
### This file is an example authorization file for svnserve.
...
[groups]
# harry_and_sally = harry,sally
# harry_sally_and_joe = harry,sally,&joe
default = seunghyun

# [/foo/bar]
# harry = rw
# &joe = r
# * =

[/]
@default = rw

...
```

## 5. SVN 서버 권한설정

```console

# vi /var/opt/svn/test-repo/conf/svnserve.conf
### This file controls the configuration of the svnserve daemon, if you
...
[general]
anon-access = none
auth-access = write
password-db = passwd
authz-db = authz
```

호스트OS에서 Volume에 직접 접근해서 SVN 설정(3번 ~ 5번)을 직접 수정하셔도 됩니다.

간단하게 SVN 서버 구성이 완료되었습니다.
***
### 설치환경
1. HostOS: Windows10 Pro 1809
2. Docker: 2.0.0.3
3. Image: garethflowers/svn-server:1.2 (https://hub.docker.com/r/garethflowers/svn-server/)