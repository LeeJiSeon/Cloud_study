# 쿠버네티스 아키텍처

## 1. 마스터 & 노드
![masternode](https://mblogthumb-phinf.pstatic.net/MjAxOTA3MzFfMTU5/MDAxNTY0NTQzODk3MDUy.TFrMWLC3yIzhfmhKwfeDG_irYy_E6a513bCH9hoyNswg.qgbONh3Du75bGC--Qr4qaao8bFS-suZVJMs3mr1bIDcg.PNG.shakey7/image.png?type=w800 "master&node")

![masternode2](https://t1.daumcdn.net/cfile/tistory/998670455B140DE22B "master&node")

- 쿠버네티스는 클러스터(그룹화) 구조를 띄는데, 클러스터 전체를 관리하는 마스터(Kubernetes Master)와 컨테이너가 배포되는 가상 또는 물리 머신인 노드(Worker Node)로 나뉜다.
- 모듈화가 되어 있고, 기능 확장을 위해서 플러그인을 설치할 수 있는 구조로 되어 있다.

### _1) 마스터 (Kubernetes Master)_

- Kubectl 커맨드 인터페이스를 통해 세팅되는 마스터는 쿠버네티스의 설정 환경을 저장하고 노드로 이루어진 클러스터 전체를 관리한다.
- API 서버, 스케쥴러, 컨트롤러 매니저, Etcd로 구성된다.

```
[API 서버]
- 유저(User Interface, UI)로 부터의 요청이나, 마스터-노드 간의 통신을 담당한다.
- 쿠버네티스의 모든 기능들을 REST API로 제공하고 그에 대한 명령을 처리한다.

[Etcd]
- 분산형 키/밸류 스토어 오픈소스
- 노드 및 클러스터의 설정값과 상태를 저장하는 데이터베이스 역할을 한다.

[스케쥴러]
- 포드의 배포 및 서비스에 필요한 리소스를 적절한 노드에 할당한다.

[컨트롤러 매니저]
- 컨트롤러(Replica controller, Service controller, Volume Controller, Node controller 등)를 생성하고 이를 각 노드에 배포하며 이를 관리하는 역할을 한다.
- 포드의 저장 공간 조절, 동적으로 추가 또는 삭제되는 포드에 대한 라벨(Label) 할당, 복제, 여러 포드의 그룹핑 서비스 시 로드 밸런싱 등을 관리한다.

[DNS]
- 쿠버네티스는 리소스의 엔드포인트(Endpoint)를 DNS로 맵핑하고 관리한다.
- Pod나 서비스등은 IP를 배정받는데, 동적으로 생성되는 리소스이기 때문에 그 IP 주소가 그때마다 변경이 되어 리소스에 대한 위치 정보가 필요하다.
- 이러한 패턴을 Service discovery 패턴이라고 하는데, 쿠버네티스에서는 이를 내부 DNS서버에 두는 방식으로 해결하였다.
- 새로운 리소스가 생기면, 그 리소스에 대한 IP와 DNS 이름을 등록하여, DNS 이름을 기반으로 리소스에 접근할 수 있도록 한다.
```

### _2) 노드 (Worker Node)_

- 노드는 마스터에 의해 명령을 받고 실제 작업을 수행하는 서비스 컴포넌트로 컨테이너를 보유한 포드들이 배치된다.
- 포드나 컨테이너 처럼 쿠버네티스 위에서 동작하는 워크로드를 호스팅하는 역할을 한다.

```
[Kubelet]
- 마스터의 API 서버와 통신을 담당하며 수행할 명령을 받거나 노드의 상태를 마스터로 전달한다.

[Kube-proxy]
- 노드에 들어오는 네트워크 트래픽을 포드 내의 컨테이너에게 라우팅하고 노드와 마스터간의 네트워크 통신을 관리한다.

[Container Runtime (컨테이너 런타임)]
- Pod를 통해서 배포된 컨테이너를 실행하는 컨테이너 런타임이다.
- 컨테이너 런타임은 보통 도커 컨테이너를 생각하기 쉬운데, 도커 이외에도 rkt (보안이 강화된 컨테이너), Hyper container 등 다양한 런타임이 있다.

[cAdvisor]
- 각 노드에서 기동되는 모니터링 에이전트로, 노드내에서 가동되는 컨테이너들의 상태와 성능등의 정보를 수집하여, 마스터 서버의 API 서버로 전달한다.
- 이 데이터들은 주로 모니터링을 위해서 사용된다.
```



---
## _* 참고_
1. <https://m.blog.naver.com/shakey7/221600947441>
1. <https://bcho.tistory.com/1256?category=731548>
