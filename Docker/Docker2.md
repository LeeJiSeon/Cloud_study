# 도커의 특징

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

* 컨테이너 생성 시 : 기존의 이미지 레이어 위에 읽기/쓰기(read-write) 레이어를 추가한다.
  * 이미지 레이어를 그대로 사용한다.
  * 컨테이너가 실행중에 생성하는 파일이나 변경된 내용은 읽기/쓰기 레이어에 저장된다
    * 여러개의 컨테이너를 생성해도 최소한의 용량만 사용할 수 있다.
    
    
### _2) 이미지 경로_
> - 이미지는 url 방식으로 관리할 수 있다.

![Imageurl](https://subicura.com/assets/article_images/2017-01-19-docker-guide-for-beginners-1/image-url.png "이미지 url 방식")

```
ubuntu 14.04 이미지 : docker.io/library/ubuntu:14.04 또는 docker.io/library/ubuntu:trusty
-> docker.io/library는 생략가능하여 ubuntu:14.04 로 사용할 수 있다. 
```
* 이 방식은 이해하기 쉽고 편리하게 사용할 수 있으며 태그 기능을 잘 이용하면 테스트나 롤백도 쉽게 할 수 있다.

### _3) Dockerfile_

```
# vertx/vertx3 debian version
FROM subicura/vertx3:3.3.1
MAINTAINER chungsub.kim@purpleworks.co.kr

ADD build/distributions/app-3.3.1.tar /
ADD config.template.json /app-3.3.1/bin/config.json
ADD docker/script/start.sh /usr/local/bin/
RUN ln -s /usr/local/bin/start.sh /start.sh

EXPOSE 8080
EXPOSE 7000

CMD ["start.sh"]
```
* 도커는 이미지를 만들기 위해 `Dockerfile`이라는 파일에 자체 DSL(Domain-specific language)언어를 이용하여 이미지 생성 과정을 적는다.
  * `FROM` : 새로운 이미지를 생성할 때 기반으로 사용할 이미지 지정
  * `RUN` : 이미지를 생성할 때 실행할 코드 지정
  * `WORKDIR` : 작업 디렉토리 지정, 해당 디렉토리가 없으면 새로 생성
  * `COPY` : 파일이나 폴더를 이미지에 복사
  * `ADD` : `COPY`와 동일한 기능을 하지만 복사한 파일의 압축을 푼다는 차이가 있음
  * `ENV` : 이미지에서 사용할 환경 변수 값 지정
  * `ENTRYPOINT` : 컨테이너를 구동할 때 실행할 명령어 지정
* 이 파일은 소스와 함께 버전관리 된다. 

### _4) Docker Hub_
---

### _* 참고_
1. <https://bcho.tistory.com/805?category=731548>
1. <https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html#%EB%8F%84%EC%BB%A4%EC%9D%98-%EC%97%AD%EC%82%AC>
1. <https://javacan.tistory.com/entry/docker-start-7-create-image-using-dockerfile>
