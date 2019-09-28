# Docker Study

>[Slipp 스터디](https://www.slipp.net/wiki/pages/viewpage.action?pageId=41582977)에서 Docker&Kubernetes 수업을 하며 정리  
참고한 책 : [도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문](http://aladin.co.kr/shop/wproduct.aspx?ItemId=186771560)

## 1. 도커 스웜

### 1.1 스웜이란?

다수의 호스트에 여러 도커 호스트를 클러스터로 묶어서 관리해주는 컨테이너 오케스트레이션.

>Docker In Docker (dind)  
도커 호스트 역할을 하는 컨테이너를 여러개 실행하여 그 컨테이너 안에서 도커를 실행하는 방법.  
실행되는 컨테이너 만큼만 메모리를 소비하여 리소스 측면의 장점이 있음.  
호스트 도커에서 관리하는 네트워크를 이용.

### 1.2 스웜의 구성요소

* 스웜

* 서비스 
 어플리케이션을 구성하는 일부 컨테이너를 제어하기 위한 단위

* 스택
하나 이상의 서비스를 그룹으로 묶은 단위. 애플리케이션 전체 구성을 정의. 
서비스로는 애플리케이션 이미지를 하나밖에 다루지 못하므로 여러 서비스가 협조해 동작할 수 있도록 서비스를 다룰 수 있다.

## 2. 스웜 클러스터 구축(week1-5)

### 2.1 도커 컴포즈 실행

여러 대의 도커 호스트로 스윔 클러스터를 구축해 보자. (dind 환경) 
* registry : 도커 레지스트리. 실무에서는 보통 도커허브/인하우스 레지스트리 사용

* manager : 스웜 클러스터 제어, 여러 대 실행되는 도커 호스트에 서비스가 담긴 컨테이너 배치

* worker : 컨테이너 3대로 구성. 실습을 위해 http를 사용해야하므로 command 요소 추가(--insecure-registry registry:5000)

### 2.2 스웜 역할 할당

* swarm init 실행 으로 manager 마스킹, join 토큰 생성

    ```sh
    docker container exec -it manager docker swarm init
    ```

* join 토큰으로 worker 등록

    ```sh
    docker container exec -it worker01 docker swarm join --token SWMTKN-1-5yfv2puvtlgv0p0vpv9pazkmglvgexgmm7pxxx4va2csalvj4y-322qhy2qm893jrf0mxtrct9bf 172.19.0.3:2377
    ```

* docker node 확인

    ```sh
    docker container exec -it manager docker node ls
    ```

* 도커 이미지 태그

    ```sh
    # 레지스트리 호스트 연결을 위해 localhost:5000을 붙인다. 
    docker image tag example/echo:latest localhost:5000/example/echo:latest
    ```

* 도커 레지스트리에 이미지 등록
    외부 도커에서 빌드한 이미지는 레지스트리를 통해서만 안쪽 도커에서 사용 가능.

    ```sh
    docker image push localhost:5000/example/echo:latest
    ```

### 2.3 컨테이너 배포

* manager 컨테이너에서 서비스 생성

    ```sh
    docker container exec -it manager docker service create --replicas 1 --publish 8000:8080 --name echo registry:5000/example/echo:latest
    # service 목록 확인
    docker container exec -it manager docker service ls
    ```

* 서비스 제어

    ```sh
    # 서비스가 제어하는 레플리카 증가
    docker container exec -it manager docker service scale echo=6
    # 컨테이너 확인
    docker container exec -it manager docker service ps echo | grep Runnin
    # 배포된 서비스 삭제
    docker container exec -it manager doker service rm echo
    ```

### 2.4 스택
* overlay 네트워크

    > 스택을 사용해 배포된 서비스 그룹은 같은 네트워크에 배치되는 overlay 네트워크에 속한다. 따라서 서로 다른 호스트에 위치한 컨테이너끼리 통신할 수 있다.

    ```sh
    # overlay 네트워크 구성
    docker container exec -it manager docker network create --driver=overlay --attachable ch03
    ```

* 스택 생성

    ``` sh
    # Nginx를 프론트엔드로, 백엔드 API의 리버스 프록시를 맡기는 구성의 수택 생성(ch03-webapi.yml)
    # -c 옵션으로 스택 정의 파일 경로 지정
    docker container exec -it manager docker stack deploy -c /stack/ch03-webapi.yml echo
    ```

* 스택 확인

    ```sh
    # 배포된 스택 서비스 목록 확인
    docker container exec -it manager docker stack services echo
    # 스택에 배포된 컨테이너 확인
    docker container exec -it manager docker stack ps echo
    # 스택 삭제하기
    docker container exec -it manager docker stack rm echo
    ```

* visualizer를 사용해 컨테이너 배치 시각화하기(visualizer.yml)
    
    ```sh
    docker container exec -it manager docker stack deploy -c /stack/visualizer.yml visualizer
    ```

### 2.5 스웜 클러스터 외부에서 서비스 사용하기

manager 컨테이너는 포트 포워딩을 통해 호스트에서 접근이 가능하지만 다른 서비스 들은 여러 컨테이너가 여러 노드에 흩어져 배치해 있기 때문에 호스트에서 포트포워딩을 통해 접근할 수 없다.
프록시 서버를 이용해 외부 트래픽이 서비스에 접근 할 수 있다.

* 프록시 서버가 서비스를 찾을 수 있도록 nginx 환경변수에 SERVICE_PORT=80 추가

* 스택 echo 배포 (ch03-webapi.yml)

* HAProxy 프록시 서버 실행(ingress.yml)

    ```sh
    docker container exec -it manager docker stack deploy -c /stack/ingress.yml ingress
    ```