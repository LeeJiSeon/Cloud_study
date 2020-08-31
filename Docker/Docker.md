# Docker 도커

## 1. 도커란?

### _* 도커 (Docker)_
> - 컨테이너 기반의 오픈소스 가상화 플랫폼
* 하나의 서버에 여러개의 컨테이너를 실행하면 서로 영향을 미치지 않고 독립적으로 실행된다.

### _* 컨테이너 (Container)_
> - 격리된 공간에서 프로세스가 동작하는 기술
* 가상화 기술의 하나지만, 기존의 방식과는 차이가 있다.

### _* VM vs Docker_
> - Docker는 Virtual Machine과 유사한 기능을 가지지만, 방식에는 차이가 있다.  
> - Docker는 VM보다 가볍고 빠르다.

* **VM (virtual machine)** : 하드웨어 가상화
  * 환경 테스팅 중심
  * 리눅스나 윈도우 등 다양한 종류의 os를 설치할 수 있다.
  * Host OS(우리가 사용하는 OS)위에 새로운 Guest OS를 설치한다.

* **Docker** : container isolation
  * 배포 중심
  * Guest OS를 새로 설치하지 않고 Host OS에 없는 것만 추가적으로 받아온다.
  * Container 자체에는 Kernel등의 OS 이미지가 들어가 있지 않다.  
  * Kernel은 Host OS를 그대로 사용하되, Host OS와 Container의 OS의 다른 부분만 Container 내에 같이 Packing된다.
  * Container에서 명령어를 수행하면 실제로 Host OS에서 명령어가 실행된다.
  * Host OS의 프로세스 공간을 공유한다.

![VmDocker](https://subicura.com/assets/article_images/2017-01-19-docker-guide-for-beginners-1/vm-vs-docker.png "vm과 docker 비교")

---

### _* 이미지 (Image)_ 
> - 컨테이너 실행에 필요한 파일과 설정값 등을 포함하고 있는 것  
> - Immutable : 상태값을 가지지 않고 변하지 않는다.

```
- 컨테이너는 이미지를 실행한 상태라고 볼 수 있다.
- 추가되거나 변하는 값은 컨테이너에 저장된다.
- 같은 이미지에서 여러 개의 컨테이너를 생성할 수 있다.
- 컨테이너의 상태가 바뀌거나 컨테이너가 삭제되더라도 이미지는 변하지 않고 그대로 남아있다.
```
* 도커 이미지는 Docker hub에 등록하거나 Docker Registry 저장소를 직접 만들어 관리할 수 있다. 

![DockerImage](https://subicura.com/assets/article_images/2017-01-19-docker-guide-for-beginners-1/docker-image.png "도커 이미지")

---

## 2. 도커의 방식

### _1) 레이어 저장방식_
```
도커 이미지는 컨테이너를 실행하기 위한 모든 정보를 가지고 있기 때문에 보통 용량이 수백메가MB에 이른다.
처음 이미지를 다운받을 땐 크게 부담이 안되지만 파일 하나를 추가했을 때 이미지를 다시 다운받는다면 비효율적일 수 밖에 없다.
```
_**-> 레이어(layer)와 유니온 파일 시스템을 이용하여 여러 개의 레이어를 하나의 파일시스템으로 사용할 수 있다.**_

* 이미지는 여러 개의 읽기전용(read only) 레이어로 구성된다.
* 파일이 추가되거나 수정되면 새로운 레이어가 생성된다.

![Layer](https://subicura.com/assets/article_images/2017-01-19-docker-guide-for-beginners-1/image-layer.png "도커 레이어")

```
ex)
1. ubuntu 이미지 : A + B + C
2. ubuntu 이미지를 기반으로 만든 nginx 이미지 : A + B + C + nginx
3. nginx 이미지룰 기반으로 만든 webapp 이미지 : A + B + C + nginx + source
=> webapp 소스 수정 시 : source 레이어만 다운 받으면 됨
```

---

### _* 참고_
1. <https://bcho.tistory.com/805?category=731548>
1. <https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html#%EB%8F%84%EC%BB%A4%EC%9D%98-%EC%97%AD%EC%82%AC>
