---
title: "SVN Commit Message 수정하기"
date: 2019-05-01
tags:
  - Subversion
  - SVN
  - edit commit message
  - edit commit comment
---

사내에서 SVN를 관리하다보면 다음과 같은 상황이 발생합니다.
* Commit 메시지를 미작성 혹은 굉장히 부실하게 작성한 경우
* Commit한 메시지를 타인이 이해하기 힘든 경우

정상적인 상황은 아니지만 어쨌든 발생하기 때문에 해결해야 합니다.  
그리고 Commit 메시지를 수정하는 일은 위험한 일이기 때문에 ```특정 사용자(관리자)```에게만 부여하려고 합니다.  
* **특정 사용자에게 Commit 메시지 수정권한 부여**

먼저 Commit 메시지 수정을 위해서는 SVN서버의 hooks폴더에 ```pre-revprop-change```파일이 있어야 합니다.

1\. svn repository의 hooks폴더로 이동
---

경로는 아래와 다를 수 있습니다.
```shell
/   # cd /var/opt/svn/test-repo/hooks
/var/opt/svn/test-repo/hooks #
```

2\. pre-revprop-change파일 생성
---

기본적으로 ```pre-revprop-change.tmpl``` 템플릿 파일이 있습니다.  
파일이 있는 경우 아래와 같이 복사하기로 진행해 주셔도 됩니다.
```shell
/var/opt/svn/test-repo/hooks # ls -al | grep pre-revprop-change
-rwxr-xr-x    1 root     root          3437 May  1 11:35 pre-revprop-change.tmpl
/var/opt/svn/test-repo/hooks # cp pre-revprop-change.tmpl pre-revprop-change
/var/opt/svn/test-repo/hooks # ls -al | grep pre-revprop-change
-rwxr-xr-x    1 root     root          3437 May  1 12:21 pre-revprop-change
-rwxr-xr-x    1 root     root          3437 May  1 11:35 pre-revprop-change.tmpl
```
  
```pre-revprop-change.tmpl```파일이 없는 경우 새로 만들어 보겠습니다.
```shell
/var/opt/svn/test-repo/hooks # vi pre-revprop-change
```
아래 코드를 입력하고 저장해주세요.
```
#!/bin/sh

REPOS="$1"
REV="$2"
USER="$3"
PROPNAME="$4"
ACTION="$5"

if [ "$ACTION" = "M" -a "$PROPNAME" = "svn:log" ]; then exit 0; fi

echo "Changing revision properties other than svn:log is prohibited" >&2
exit 1
```

3\. pre-revprop-change파일 권한 수정하기
---

```shell
/var/opt/svn/test-repo/hooks # chmod 755 pre-revprop-change
```
축하합니다. 모든 svn사용자가 Commit 메시지를 수정 할 수 있는 상태가 되었습니다.

4\. 특정 사용자에게 Commit 메시지 수정권한 부여하기
---

```shell
/var/opt/svn/test-repo/hooks # vi pre-revprop-change
```

#Allow user list 부터 if문까지 수정해줍니다.

```shell
#!/bin/sh

REPOS="$1"
REV="$2"
USER="$3"
PROPNAME="$4"
ACTION="$5"

### 특정 사용자에게만 접근을 허용하기 위한 수정 코드 ###
# Allow user list
ALLOW_USERS=("user01" "user02" "user03") # Commit 메시지를 수정할 유저를 Array에 추가해준다.
is_allow_user () {
    for (( i = 0 ; i < ${#ALLOW_USERS[@]} ; i++ )) ; do
        if [ "$USER" = ${ALLOW_USERS[$i]} ]; then return 0; fi
    done
    return 1
}

if [ "$ACTION" = "M" -a "$PROPNAME" = "svn:log" ] && is_allow_user; then exit 0; fi
#####################################################

echo "Changing revision properties other than svn:log is prohibited" >&2
exit 1
```
축하합니다. 특정 svn사용자만 Commit 메시지를 수정 할 수 있는 상태가 되었습니다.  

**[해설]**  
1. 특정 수정 요청에 대해서 ```svn서버```가 ```pre-revprop-change```파일에 ```[REPOS, REV, USER, PROPNAME, ACTION]```을 Argument로 전달하고 실행시킨다.  
2. ```exit 0```이면 수정 가능, ```exit 1```이면 수정 불가능이다.
3. Argument로 받는 ```User```를 비교하는 로직을 추가해서 특정 사용자만 Commit 메시지 수정이 가능하게 만들었다.


**Commit 메시지를 수정하는 일은 신중하고, 조심해야되는 위험한 일입니다.** 불가피한 경우 외에는 가급적 사용하지 않는게 좋습니다.


[보너스] Commit 후 Author 수정하기
---

```shell
/var/opt/svn/test-repo/hooks # vi pre-revprop-change

#!/bin/sh

REPOS="$1"
REV="$2"
USER="$3"
PROPNAME="$4"
ACTION="$5"

if [ "$ACTION" = "M" -a "$PROPNAME" = "svn:log" ]; then exit 0; fi
### author 수정 부분 추가 ###
if [ "$ACTION" = "M" -a "$PROPNAME" = "svn:author" ]; then exit 0; fi 


echo "Changing revision properties other than svn:log is prohibited" >&2
exit 1
```
