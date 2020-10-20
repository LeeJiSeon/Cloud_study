# 쿠버네티스 아키텍처

## 1. 마스터 & 노드
> - 중앙(Maste)에 API 서버와 상태 저장소를 두고 각 서버(Node)의 에이전트(kubelet)와 통신하는 단순한 구조

![masternode2](https://subicura.com/assets/article_images/2019-05-19-kubernetes-basic-1/master-node.png "master&node")

![masternode](https://mblogthumb-phinf.pstatic.net/MjAxOTA3MzFfMTU5/MDAxNTY0NTQzODk3MDUy.TFrMWLC3yIzhfmhKwfeDG_irYy_E6a513bCH9hoyNswg.qgbONh3Du75bGC--Qr4qaao8bFS-suZVJMs3mr1bIDcg.PNG.shakey7/image.png?type=w800 "master&node")

- 쿠버네티스는 클러스터(그룹화) 구조를 띄는데, 클러스터 전체를 관리하는 마스터(Kubernetes Master)와 컨테이너가 배포되는 가상 또는 물리 머신인 노드(Worker Node)로 나뉜다.
- 모듈화가 되어 있고, 기능 확장을 위해서 플러그인을 설치할 수 있는 구조로 되어 있다.

<br>

- 모든 명령은 마스터의 API 서버를 호출하고 노드는 마스터와 통신하면서 필요한 작업을 수행한다.
- 특정 노드의 컨테이너에 명령하거나 로그를 조회할 때도 노드에 직접 명령하는 게 아니라 마스터에 명령을 내리고 마스터가 노드에 접속하여 대신 결과를 응답한다.

### _1) 마스터 (Kubernetes Master)_

- Kubectl 커맨드 인터페이스를 통해 세팅되는 마스터는 쿠버네티스의 설정 환경을 저장하고 노드로 이루어진 클러스터 전체를 관리한다.
- 관리자만 접속할 수 있도록 보안 설정을 해야 한다.
- 마스터 서버가 죽으면 클러스터를 관리할 수 없기 때문에 보통 3대를 구성해 안정성을 높인다.
- 다양한 모듈이 확장성을 고려해 기능별로 쪼개져 있다.
- 개발 환경이나 소규모 환경에선 마스터와 노드를 분리하지 않고 같은 서버에 구성하기도 한다.
- API 서버, 스케쥴러, 컨트롤러 매니저, Etcd로 구성된다.

![master](https://subicura.com/assets/article_images/2019-05-19-kubernetes-basic-1/kubernetes-master.png "master")

```
[API 서버] : 모든 요청을 처리하는 마스터의 핵심 모듈

- 유저(User Interface, UI)로 부터의 요청이나, 마스터-노드 간의 통신을 담당한다.
- 쿠버네티스의 모든 기능들을 REST API로 제공하고 그에 대한 명령을 처리한다.

- kubectl의 요청뿐 아니라 내부 모듈의 요청도 처리하며 권한을 체크하여 요청을 거부할 수 있다.
- 실제로 하는 일은 원하는 상태를 key-value 저장소에 저장하고 저장된 상태를 조회하는 매우 단순한 작업이다.
- Pod을 노드에 할당하고 상태를 체크하는 일은 다른 모듈로 분리되어 있다.
- 노드에서 실행 중인 컨테이너의 로그를 보여주고 명령을 보내는 등의 디버거 역할도 수행한다.

[Etcd] : 분산 데이터 저장소

- 분산형 키/밸류 스토어 오픈소스
-> RAFT 알고리즘을 이용한 key-value 저장소
-> 여러 개로 분산하여 복제할 수 있기 때문에 안정성이 높고 속도가 빠르다.

- 노드 및 클러스터의 설정값과 상태를 저장하는 데이터베이스 역할을 한다.
-> 나머지 모듈은 stateless하게 동작하기 때문에 etcd만 잘 백업해두면 언제든지 클러스터를 복구할 수 있다.

- watch 기능이 있어 어떤 상태가 변경되면 바로 체크하여 로직을 실행할 수 있다.
- 오직 API서버와 통신하고 다른 모듈은 API서버를 거쳐 etcd 데이터에 접근한다.

----------

- API 서버는 요청을 받으면 etcd 저장소와 통신할 뿐 실제로 상태를 바꾸는 건 스케줄러와 컨트롤러이다.
- 현재 상태를 모니터링하다가 원하는 상태와 다르면 각자 맡은 작업을 수행하고 상태를 갱신한다.

[스케쥴러]
- 할당되지 않은 Pod을 여러가지 조건(필요한 자원, 라벨)에 따라 적절한 노드 서버에 할당해주는 모듈이다.
- 포드의 배포 및 서비스에 필요한 리소스를 적절한 노드에 할당한다.

[컨트롤러 매니저]
- 컨트롤러(Replica controller, Service controller, Volume Controller, Node controller 등)를 생성하고 이를 각 노드에 배포하며 이를 관리하는 역할을 한다.
- 포드의 저장 공간 조절, 동적으로 추가 또는 삭제되는 포드에 대한 라벨(Label) 할당, 복제, 여러 포드의 그룹핑 서비스 시 로드 밸런싱 등을 관리한다.

1. 큐브 컨트롤러 (kube-controller-manager)
- 쿠버네티스에 있는 거의 모든 오브젝트의 상태를 관리한다.
- 오브젝트 별로 철저하게 분업화되어 있다.
ex) Deployment -> ReplicaSet 생성
    ReplicaSet -> Pod 생성
    Pod -> 스케줄러가 관리
    
2. 클라우드 컨트롤러 (cloud-controller-manager)
- AWS, GCE, Azure 등 클라우드에 특화된 모듈이다.
- 노드를 추가/삭제하고 로드 밸런서를 연결하거나 볼륨을 붙일 수 있다.
- 각 클라우드 업체에서 인터페이스 맞춰 구현하면 되기 때문에 확장성이 좋다.

----------

[DNS]
- 쿠버네티스는 리소스의 엔드포인트(Endpoint)를 DNS로 맵핑하고 관리한다.
- Pod나 서비스등은 IP를 배정받는데, 동적으로 생성되는 리소스이기 때문에 그 IP 주소가 그때마다 변경이 되어 리소스에 대한 위치 정보가 필요하다.
- 이러한 패턴을 Service discovery 패턴이라고 하는데, 쿠버네티스에서는 이를 내부 DNS서버에 두는 방식으로 해결하였다.
- 새로운 리소스가 생기면, 그 리소스에 대한 IP와 DNS 이름을 등록하여, DNS 이름을 기반으로 리소스에 접근할 수 있도록 한다.
```

### _2) 노드 (Worker Node)_

- 노드는 마스터에 의해 명령을 받고 실제 작업을 수행하는 서비스 컴포넌트로 컨테이너를 보유한 포드들이 배치된다.
- 포드나 컨테이너 처럼 쿠버네티스 위에서 동작하는 워크로드를 호스팅하는 역할을 한다.
- 실제 컨테이너들이 생성되는 곳으로 수백, 수천대로 확장할 수 있다.
- 각각의 서버에 라벨을 붙여 사용목적(GPU 특화, SSD 서버 등)을 정의할 수 있다.

![node](https://subicura.com/assets/article_images/2019-05-19-kubernetes-basic-1/kubernetes-node.png "node")

```
[Kubelet]
- 노드에 할당된 Pod의 생명주기를 관리한다.
- 마스터의 API 서버와 통신을 담당하며 수행할 명령을 받거나 노드의 상태를 마스터로 전달한다.
-> Pod을 생성하고 Pod안의 컨테이너에 이상이 없는지 확인하면서 주기적으로 마스터에 상태를 전달한다.
-> API 서버의 요청을 받아 컨테이너의 로글르 전달하거나 특정 명령을 대신 수행하기도 한다.

[Kube-proxy]
- Pod으로 연결되는 네트워크를 관리한다.
-> 노드에 들어오는 네트워크 트래픽을 포드 내의 컨테이너에게 라우팅하고 노드와 마스터간의 네트워크 통신을 관리한다.
- TCP, UDP, SCTP 스트림을 포워딩하고 여러 개의 Pod을 라운드로빈 형태로 묶어 서비스를 제공할 수 있다.

[Container Runtime (컨테이너 런타임)]
- Pod를 통해서 배포된 컨테이너를 실행하는 컨테이너 런타임이다.
- 컨테이너 런타임은 보통 도커 컨테이너를 생각하기 쉬운데, 도커 이외에도 rkt (보안이 강화된 컨테이너), Hyper container 등 다양한 런타임이 있다.

[cAdvisor]
- 각 노드에서 기동되는 모니터링 에이전트로, 노드내에서 가동되는 컨테이너들의 상태와 성능등의 정보를 수집하여, 마스터 서버의 API 서버로 전달한다.
- 이 데이터들은 주로 모니터링을 위해서 사용된다.
```

### _3) Kubectl_

- API 서버는 json 또는 protobuf 형식을 이용한 http 통신을 지원한다.
- 이 방식을 그대로 쓰면 불편하기 때문에 `kubectl`이라는 명령행 도구를 사용한다.


## 2. Pod 생성 과정

![pod](https://subicura.com/assets/article_images/2019-05-19-kubernetes-basic-1/create-replicaset.png)

- 관리자가 애플리케이션을 배포하기 위해 ReplicaSet을 생성하면 위와 같은 과정을 거쳐 Pod를 생성한다.
- 각 모듈은 서로 통신하지 않고 오직 API Server와 통신한다.

### _kubectl_
- ReplicaSet명세를 yml파일로 정의하고 kubectl도구를 이용하여 API Server에 명령을 전달한다.
- API Server는 새로운 ReplicaSet Object를 etcd에 저장한다.

### _Kube Controller_
- Kube Controller에 포함된 ReplicaSet Controller가 ReplicaSet을 감시하다가 ReplicaSet에 정의된 Label Selector 조건을 만족하는 Pod이 존재하는지 체크한다.
- 해당하는 Label의 Pod이 없으면 ReplicaSet의 Pod 템플릿을 보고 새로운 Pod(no assign)을 생성한다.
-> 생성은 역시 API Server에 전달하고 API Server는 etcd에 저장한다.

### _Scheduler_
- Scheduler는 할당되지 않은(no assign) Pod이 있는지 체크한다.
- 할당되지 않은 Pod이 있으면 조건에 맞는 Node를 찾아 해당 Pod을 할당한다.

### _Kubelet_
- Kubelet은 자신의 Node에 할당되었지만 아직 생성되지 않은 Pod이 있는지 체크한다.
- 생성되지 않은 Pod이 있으면 명세를 보고 Pod을 생성한다.
- Pod의 상태를 주기적으로 API Server에 전달한다.

---
## _* 참고_
1. <https://m.blog.naver.com/shakey7/221600947441>
1. <https://bcho.tistory.com/1256?category=731548>
1. <https://subicura.com/2019/05/19/kubernetes-basic-1.html#%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98>
