---
title: "무작정 쿠버네티스(Kubernetes) 해보기 02"
date: 2019-08-10
tags:
  - Kubernetes
  - k8s
  - minikube
  - 쿠버네티스
  - 미니쿠베
  - 무작정 쿠버네티스 해보기
---

***********
온프레미스(On-premise)베이스의 제품을 클라우드 컨테이너기반의 제품으로 적용(테스트)시키는 개인적인 경험에 대한 시리즈입니다. 가볍게 읽어주시면 감사하겠습니다.

***********

시리즈
---
0. ~~쿠버네티스에 대한 개념 잡기 (추가예정)~~
1. MiniKube 설치하기 (Window10 pro)
2. **공식 홈페이지 예제 실행하기(+.yaml)**
3. ~~제품 컨테이너 구동하기~~
4. ~~Pod으로 단일 컨테이너 구동하기~~
5. (추가예정)

# 공식 홈페이지 예제 실행하기(+.yaml)
쿠버네티스 공식홈페이지(https://kubernetes.io/ko/docs/setup/learning-environment/minikube/)의 빠른실행 예제(CLI)를 실행하고, 같은 내용을 yaml 파일로 작성해서 실행을 해보겠습니다.

* 1~6 빠른실행 예제 따라하기
* 7~8 YAML파일로 바꿔서 실행하기

1\. Deployment 만들기
---
```console
C:\Users\kimse\minikube_test> kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.10 --port=8080
deployment.apps "hello-minikube" created
```

2\. Service 만들기
---
```console
C:\Users\kimse\minikube_test> kubectl expose deployment hello-minikube --type=NodePort
service/hello-minikube exposed
```

3\. kubectl로 오브젝트 확인하기
---
```console
C:\Users\kimse\minikube_test> kubectl get pod
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-856979d68c-5lg8q   1/1     Running   0          85m

C:\Users\kimse\minikube_test> kubectl get service
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-minikube   NodePort    10.109.95.124   <none>        8080:31000/TCP   84s
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP          11h

C:\Users\kimse\minikube_test> kubectl get deployment
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
hello-minikube   1/1     1            1           85m

C:\Users\kimse\minikube_test> kubectl get replicaset
NAME                        DESIRED   CURRENT   READY   AGE
hello-minikube-856979d68c   1         1         1       86m
```
`kubectl get all` 명령줄을 사용하면 위의 결과를 한번에 출력한다. (pod + service + deployment + replicaset)

4\. 서비스 호출하기
---
서비스의 URL 얻기
```console
C:\Users\kimse\minikube_test> minikube service hello-minikube --url
* http://192.168.236.172:31000
```
* 출력 결과의 port번호는 `kubectl get service`에서 8080:31000/TCP에서 확인된 포트와 같다.
* IP주소는 `minikube ip`를 사용하면 얻는 주소와 같다.

브라우저 접속
```console
C:\Users\kimse\minikube_test> minikube service hello-minikube
```
* `--url`을 빼고 실행하면 기본 브라우저로 접속한다.

hello-minikube 호출 결과
```yaml
Hostname: hello-minikube-856979d68c-5lg8q

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008

Request Information:
	client_address=172.17.0.1
	method=GET
	real path=/
	query=
	request_version=1.1
	request_scheme=http
	request_uri=http://192.168.236.172:8080/

Request Headers:
	accept=text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
	accept-encoding=gzip, deflate
	accept-language=ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
	connection=keep-alive
	dnt=1
	host=192.168.236.172:31000
	upgrade-insecure-requests=1
	user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.100 Safari/537.36

Request Body:
	-no body in request-
```

5\. 리소스 삭제
---
```console
C:\Users\kimse\minikube_test> kubectl delete services hello-minikube
service "hello-minikube" deleted

C:\Users\kimse\minikube_test> kubectl delete deployment hello-minikube
deployment.extensions "hello-minikube" deleted
```
***
# YAML파일로 빠른실행 예제 재현하기

6\. YAML파일로 Deployment 만들기
---
Deployment 오브젝트 기술하기 (YAML파일 'hello-minikube-deployment.yml' 작성)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-minikube
  labels:
    run: hello-minikube
spec:
  replicas: 1
  selector:
    matchLabels:
      run: hello-minikube
  template:
    metadata:
      labels:
        run: hello-minikube
    spec:
      containers:
      - name: hello-minikube
        image: k8s.gcr.io/echoserver:1.10
        ports:
        - containerPort: 8080
```
위와 내용은 같습니다. 주석만 추가되었습니다.
```yaml
#오브젝트 생성하기 위해 사용하고 있는 쿠버네티스 API버전
apiVersion: apps/v1
#오브젝트 종류
kind: Deployment
#오브젝트를 구별 해 줄수 있는 정보
metadata:
  #오브젝트 이름
  name: hello-minikube
  #사용자에게 오브젝트의 속성을 제공하기 위해 임의로 붙이는 속성 값
  labels:
    run : hello-minikube
#오브젝트에게 원하는 상태
spec:
  #대상 리소스(replicas)를 1개를 띄워라
  replicas: 1
  #대상 리소스 정보(라벨정보가 'run: hello-minikube'인 리소스)
  selector:
    matchLabels:
      run: hello-minikube
  #대상 리소스에 대한 스펙
  template:
    #대상 리소스의 메타정보
    metadata:
      labels:
        run: hello-minikube
    #대상 리소스에게 원하는 상태
    spec:
      #컨테이너를 구동해라
      containers:
        #컨테이너 이름
      - name: hello-minikube
        #컨테이너를 구동할 이미지정보
        image: k8s.gcr.io/echoserver:1.10
        #컨테이너가 오픈할 포트
        ports:
        - containerPort: 8080
```
* 오브젝트 기술에 대한 자세한 정보 링크: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md

Deployment YAML파일 실행하기
```console
C:\Users\kimse\minikube_test> kubectl apply -f hello-minikube-deployment.yml
deployment.apps/hello-minikube created
```
7\. YAML파일로 Service 만들기
---
Deployment 오브젝트 기술하기 (YAML파일 'hello-minikube-service.yml' 작성)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-minikube
  labels:
    run: hello-minikube
spec:
  selector:
    run: hello-minikube
  ports:
  - protocol: TCP
    port: 18080
    targetPort: 8080
    nodePort: 32000
  type: NodePort
```
위와 내용은 같습니다. 주석만 추가되었습니다.
```yaml
#오브젝트 생성하기 위해 사용하고 있는 쿠버네티스 API버전
apiVersion: v1
#오브젝트 종류
kind: Service
#오브젝트를 구별 해 줄수 있는 정보
metadata:
  #오브젝트 이름
  name: hello-minikube
  #사용자에게 오브젝트의 속성을 제공하기 위해 임의로 붙이는 속성 값
  labels:
    run: hello-minikube
#오브젝트에게 원하는 상태
spec:
  #대상 리소스 정보(라벨정보가 'run: hello-minikube'인 리소스)
  selector:
    run: hello-minikube
  #대상 리소스에 대한 포트정보
  ports:
    #프로토콜 타입
  - protocol: TCP
    #서비스를 직접 통해서 접근하는 포트 (이 예제에서는 동작이 없다.)
    port: 18080
    #컨테이너에 오픈되어 있는 포트
    targetPort: 8080
    #노드에 직접 오픈할 포트 (이 예제에서 사용하는 접근포트)
    nodePort: 32000
  #서비스 오브젝트의 타입
  type: NodePort
```

Service YAML파일 실행하기
```console
C:\Users\kimse\minikube_test> kubectl apply -f hello-minikube-service.yml
service/hello-minikube created
```

8\. 호출하기
---
```console
C:\Users\kimse\minikube_test> minikube service hello-minikube --url
* http://192.168.236.172:32000
```
* 포트번호가 32000으로 nodePort로 고정되어 있는게 확인됩니다.

호출결과
```yaml
Hostname: hello-minikube-856979d68c-75p8g

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008

Request Information:
	client_address=172.17.0.1
	method=GET
	real path=/
	query=
	request_version=1.1
	request_scheme=http
	request_uri=http://192.168.236.172:8080/

Request Headers:
	accept=text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
	accept-encoding=gzip, deflate
	accept-language=ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
	connection=keep-alive
	dnt=1
	host=192.168.236.172:32000
	upgrade-insecure-requests=1
	user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.100 Safari/537.36

Request Body:
	-no body in request-
```

공식홈페이지 빠른실행 예제를 따라하고, 같은 내용을 YAML문서를 작성해서 실행해 보았습니다.

***

## 실행중 발생한 오류 및 해결방법
yaml문서 실행 중 오류발생
```console
C:\Users\kimse\minikube_test> kubectl apply -f hello-minikube-deployment.yml
error: SchemaError(io.k8s.api.apps.v1.DaemonSetStatus): invalid object doesn't have additional properties
```
* 원인: kubectl의 버전이 YAML문서에 기술된 버전(`apiVersion: apps/v1`)을 지원하지 않기 때문에 발생했습니다.

현재 kubectl버전 확인
```console
C:\Users\kimse>kubectl version
Client Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.11", GitCommit:"637c7e288581ee40ab4ca210618a89a555b6e7e9", GitTreeState:"clean", BuildDate:"2018-11-26T14:38:32Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"windows/amd64"}
Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.2", GitCommit:"f6278300bebbb750328ac16ee6dd3aa7d3549568", GitTreeState:"clean", BuildDate:"2019-08-05T09:15:22Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
```
* `Client Version`의 `Minor버전이 "10"`으로 예제의 apiVersion을 지원하지 않는것 같습니다.

* 해결방법: kubectl를 업데이트 합니다.

1. kubectl 다운로드 - https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/windows/amd64/kubectl.exe
2. 기존의 C:\Program Files\Docker\Docker\resources\bin\kubectl.exe에 덮어씌우기 (기존파일 백업)

kubectl.exe 업데이트 후 버전확인
```console
C:\Users\kimse>kubectl version
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.0", GitCommit:"e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529", GitTreeState:"clean", BuildDate:"2019-06-19T16:40:16Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"windows/amd64"}
Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.2", GitCommit:"f6278300bebbb750328ac16ee6dd3aa7d3549568", GitTreeState:"clean", BuildDate:"2019-08-05T09:15:22Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
```
***
## 부록
pod에 대한 yaml문서입니다. deployment의 spec부분에 들어가는 내용과 비교하면서 보시면 좋을 것 같습니다.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-minikube
  labels:
    run: hello-minikube
spec:
  containers:
  - name: hello-minikube
    image: k8s.gcr.io/echoserver:1.10
    ports:
    - containerPort: 8080
```

***
### 시스템 환경
1. HostOS: Windows10 Pro 1809
2. Minikube: 1.3.0