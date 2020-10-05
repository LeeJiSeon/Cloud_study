# 쿠버네티스 개념

## 1. Desired State
> - 관리자가 바라는 환경 (얼마나 많은 웹서버가 떠 있으면 좋은지, 몇 번 포트로 서비스하기를 원하는지 등)

![desiredstate](https://subicura.com/assets/article_images/2019-05-19-kubernetes-basic-1/desired-state.png)

- 쿠버네티스는 **현재 상태**를 모니터링하면서 관리자가 설정한 **원하는 상태**를 유지하려고 내부적으로 작업을 하는 로직을 가지고 있다.
- 따라서 관리자가 서버를 배포할 때 직접적인 동작을 명령하지 않고 상태를 선언하는 방식을 사용한다.

```
<ex>
- 명령 : nginx 컨테이너를 실행해줘 그리고 80 포트로 오픈해줘
- 선언 : 80 포트를 오픈한 nginx 컨테이너를 1개 유지해줘
```

- 명령과 선언의 차이는 CLI 명령어에서도 드러난다.
```bash
$ docker run # 명령
$ kubectl create # 상태 생성 (물론 kubectl run 명령어도 있지만 잘 사용하지 않습니다)
```
- 쿠버네티스의 핵심은 상태이며 쿠버네티스를 사용하려면 어떤 상태가 있고 어떻게 상태를 선언하는지를 알아야 한다.

## 2. 오브젝트
- 쿠버네티스는 상태를 관리하기 위한 대상을 오브젝트로 정의한다.
- 기본으로 수십 가지 오브젝트를 제공하고 새로운 오브젝트를 추가하기가 매우 쉽기 때문에 확장성이 좋다.


### _<Object Spec - YAML>_

```json
apiVersion: v1
kind: Pod
metadata:
    name: example
spec:
    containers:
    - name: busybox
        image: busybox:1.25
```
```
- apiVersion : 스크립트를 실행하기 위한 쿠버네티스 API 버전이다. 보통 v1을 사용한다.
- kind : 오브젝트의 종류를 정의한다.
- metadata : 오브젝트의 각종 메타데이터(라벨, 리소스의 이름 등)를 넣는다.
- spec : 오브젝트에 대한 상세 스펙을 정의한다.
```

- 오브젝트의 명세는 YAML파일로 정의하고 여기에 오브젝트의 종류와 원하는 상태를 입력한다.
- 명세는 생성, 조회, 삭제로 관리할 수 있기 때문에 REST API로 쉽게 노출할 수 있다.
- 접근 권한 설정도 같은 개념을 적용하여 누가 어떤 오브젝트에 어떤 요청을 할 수 있는지 정의할 수 있다.


### _1) Pod_
> - 쿠버네티스에서 배포할 수 있는 가장 작은 단위

![pod](https://subicura.com/assets/article_images/2019-05-19-kubernetes-basic-1/pod.png)


#### Object Spec
 
```json
apiVersion: v1
kind: Pod
metadata:
    name: ngnix
spec:
    containers:
    - name: nginx
      image: nginx:1.7.9
      ports:
        - containerPort: 8090
```
```
- Pod는 컨테이너를 가지고 있기 때문에 container를 정의한다.
    1. 이름 : nginx
    2. 도커 이미지 : nginx:1.7.9
    3. 컨테이너 포트 : 8090
```

- 한 개 이상의 컨테이너와 스토리지, 네트워크 속성을 가진다.
- Pod에 속한 컨테이너는 스토리지와 네트워크를 공유한다.
- Pod에 속한 컨테이너는 IP와 Port를 공유하며 서로 localhost로 접근할 수 있다.
```
<ex>
- 컨테이너 A : 8080 Port
- 컨테이너 B : 7001 Port
- A -> B : localhost:7001
- B -> A : localhost:8080
```
- 컨테이너를 하나만 사용하는 경우도 반드시 Pod로 감싸서 관리한다.

### _2) ReplicaSet_
> - Pod를 여러 개 (한 개 이상) 복제하여 관리하는 오브젝트

![replicaset](https://subicura.com/assets/article_images/2019-05-19-kubernetes-basic-1/replicaset.png)

- Pod를 생성하고 개수를 유지하려면 반드시 사용해야 한다.
- 복제할 개수, 개수를 체크할 라벨 선택자, 생성할 Pod의 설정값 (템플릿) 등을 가지고 있다.
- 직접적으로 사용하기 보다는 Deployment 등 다른 오브젝트에 의해 사용되는 경우가 많다.

### _3) Volume_
> - 저장소와 관련된 오브젝트
> - 컨테이너의 외장 디스크

- Pod가 기동할 때 디폴트로 컨테이너마다 로컬 디스크를 생성해서 기동되며 이는 영구적이지 못하다.
- 컨데이너가 리스타트 되거나 새로 배포될 때마다 Pod 설정에 따라 새롭게 정의되기 때문에 디스크에 기록된 내용이 유실된다.
- 데이터베이스와 같이 영구적으로 파일을 저장해야 하는 경우에 사용하는 스토리지이다.
- 호스트 디렉터리를 그대로 사용할 수도 있고 EBS와 같은 스토리지를 동적으로 생성하여 사용할 수도 있다.

### _4) Service_
> - 네트워크와 관련된 오브젝트

- Pod를 서비스로 제공할 때, 일반적인 분산환경에서는 여러개의 Pod를 서비스 하면서 이를 로드밸런서를 이용해 하나의 IP와 포트로 묶어서 서비스를 제공한다.
- Pod는 동적으로 생성이 되고 장애가 생기된 자동으로 리스타트 되면서 IP가 바뀌기 때문에 로드밸런서에서 Pod의 목록을 지정할 때 IP주소를 이용하는 것은 어렵다.
- 따라서 추가/삭제된 Pod목록을 로드밸런서가 유연하게 선택하기 위해 라벨과 라벨 셀렉터를 사용한다.

<br>

- 라벨 셀렉터 : 서비스를 정의할 때 어떤 Pod를 서비스로 묶을 것인지 정의하는 것
- 라벨 : Pod를 생성할 때 메타데이터 정보 부분에 정의
- 라벨 셀렉터에서 특정 라벨을 가지고 있는 Pod만 선택하여 서비스에 묶는다.

![service](https://t1.daumcdn.net/cfile/tistory/99B11D475B02D9C802)

- 위 그림은 라벨이 `myapp`인 Pod만 골라내서 서비스에 넣고, 그 Pod간에만 로드밸런싱을 통해 외부로 서비스를 제공하는 형태이다.

#### Object Spec
```json
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```
```
- spec 부분에 서비스에 대한 스펙을 정의한다.
    1. selector : 라벨 app이 myapp인 Pod만을 선택해서 서비스에 바인딩한다.
    2. ports : TCP 이용, 80 포트로 서비스 하되 서비스의 80 포트의 요청을 컨테이너의 9376 포트로 연결해서 서비스를 제공한다.
```

### _4) NameSpace_
> - 쿠버네티스 클러스터 내의 논리적인 분리단위

- Pod, Service 등은 네임스페이스별로 생성, 관리될 수 있다.
- 사용자의 권한은 네임스페이스별로 나눠서 부여할 수 있다.
- 네임스페이스별로 리소스의 할당량을 지정할 수 있다.
- 물리적인 분리가 아니기 때문에 다른 네임스페이스의 Pod 사이에 통신이 가능하다.

![namespace](https://t1.daumcdn.net/cfile/tistory/999A364D5B02D9C834)

### _5) Label_

- 쿠버네티스의 오브젝트를 선택할 때 사용된다.
- 라벨 검색 조건에 따라 특정 라벨을 가지고 있는 오브젝트만을 선택할 수 있다.
```
<ex>
- 특정 오브젝트만 배포
- 특정 오브젝트만 Service에 연결
- 특정 오브젝트에만 네트워크 접근권한 부여
```

#### Label
- metadata 섹션에 키/값 쌍으로 정의가 가능하다.
- 하나의 오브젝트에 여러 라벨을 동시에 적용할 수 있다.
```json
"metadata": {
  "labels": {
    "key1" : "value1",
    "key2" : "value2"
  }
}
```
#### Selector
- Object Spec에서 selector라고 정의하고 라벨 조건을 적는다.
- Equality based selector : 같냐, 다르냐의 조건을 이용하여 오브젝트를 선택하는 방법
```
<ex>
- environment = dev
- tier != frontend
```
- set based selector : 집함의 개념을 사용한다.
```
<ex>
- environment in (production,qa) : environment가 production 또는 qa인 오브젝트
- tier notin (frontend,backend) : tier가 frontend도 아니고 backend도 아닌 오브젝트
```


### _<쿠버네티스 배포방식>_
- 어플리케이션을 배포하기 위해 원하는 상태(desired state)를 다양한 오브젝트에 라벨을 붙여 정의(yaml)하고 API 서버에 전달하는 방식을 사용한다.

```
<ex>
"컨테이너를 2개 배포하고 80 포트로 오픈해줘"

- 위의 작업을 실행하기 위해 아래와 같은 명령을 전달해야 한다.

“컨테이너를 Pod으로 감싸고 type=app, app=web이라는 라벨을 달아줘.
type=app, app=web이라는 라벨이 달린 Pod이 2개 있는지 체크하고 없으면 Deployment Spec에 정의된 템플릿을 참고해서 Pod을 생성해줘.
그리고 해당 라벨을 가진 Pod을 바라보는 가상의 서비스 IP를 만들고 외부의 80 포트를 방금 만든 서비스 IP랑 연결해줘.”
```


---
## _* 참고_
1. <https://subicura.com/2019/05/19/kubernetes-basic-1.html>
1. <https://bcho.tistory.com/1256?category=731548>
