# Docker Study
>[Slipp 스터디](https://www.slipp.net/wiki/pages/viewpage.action?pageId=41582977)에서 Docker&Kubernates 수업을 하며 정리
>참고한 책 : [도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문](http://aladin.co.kr/shop/wproduct.aspx?ItemId=186771560)

## 1. Docker Container
### 1.1 Docker Container 생애주기
* 실행 중 : 도커 이미지 기반 컨테이너 생성 시 Dockerfile의 CMD/ENTRYPOINT에 정의된 애플리케이션이 실행되며 애플리케이션이 실행되는 상태가 컨테이너의 실행중 상태가 된다.
* 정지 상태 : 실행 중 상태의 애플리케이션이 종료되거나 사용자가 명시적으로 컨테이너를 정지한 경우 정지 상태가 된다. 디스크에 종료되던 시점의 상태가 저장되 남는다.
* 파기 상태 : 정지중인 컨테이너는 디스크를 차지하므로 불필요한 경우 파기하여 영구 제거한다.

### 1.2 Docker 명령어
#### docker image
```bash
# 생성된 docker image 조회(repository, tag, id, size, created 등)
docker image ls 
# docker image를 받아옴
docker image pull alpine:latest
# docker image를 빌드함 (-t(이미지명:태그명, 태그명 생략시 latest), -f(Dockerfile의 파일명 변경))
docker image build -f Dockerfile-test
```
#### docker container
```bash
# 도커 컨테이너 실행(-d(백그라운드 실행), -p(포트포워딩), -i(표준 입력,연결 유지), -t(유사 터미널 기능 활성화), --rm(컨테이너 정지 시 삭제))
docker container run alpine:latest
# 도커 컨테이너 조회(-q(컨테이너 아이디만 추출), -a(종료된 컨테이너 조회))
docker container ls -q
# 도커 컨테이너 종료
docker container stop alpine:latest
# 도커 컨테이너 삭제(-f(실행중인 컨테이너 삭제))
docker container rm f66f6f2013da
```

### 1.3 도커 컨테이너 오케스트레이션 시스템
* Docker Compose를 이용해 yaml 포맷으로 의존성있는 여러 컨테이너를 쉽게 관리 가능하다.
* Docker Compose가 여러 서버에 걸쳐있는 컨테이너를 관리하는 기법을 컨테이너 오케스트레이션 이라고 한다.
* 컨테이너 오케스트레이션을 통해 컨테이너 증가/감소/배치, 로드밸런싱, 배포 관리 등 다양한 장점을 활용할 수 있다.

## 2. 실습
### 2.1 도커 컴포즈로 여러 컨테이너 실행하기
* 도커 이미지 생성
* docker-compose.yml 파일 작성
```xml
version: "3" // yml파일 문법 버전
services:
  echo-service01: // 컨테이너 이름
    image: example/echo:latest // 사용할 이미지, 대신 Dockerfile 위치 지정 가능
    ports:
      - 10000:8080 // 포트 포워딩
  echo-service02:
    image: example/echo:latest
    ports:
      - 10001:8080
``` 
* 컨테이너 실행
```
# --build를 사용하면 강제로 다시 빌드 가능
docker-compose up -d
```
* 컨테이너 종료
```
docker-compose down
```
### 2.2 젠킨스 컨테이너 실행하기
* docker hub 이미지 사용
* docker-compose.yml 파일 작성
```yaml
version: "3"
services:
  master:
    container_name: master
    image: jenkinsci/jenkins:2.142-slim
    ports:
      - 8080:8080
# 호스트와 컨테이너 사이에 파일을 공유하는 디렉토리 지정
    volumes:
      - ./jenkins_home:/var/jenkins_home
```
* 컨테이너 실행 후 jenkins plugins 설치
설치를 진행하면 /var/jenkins_home 위치에 데이터가 저장되므로 컨테이너를 재시작해도 초기 설정이 유지된다
* 마스터 젠킨스용 SSH 키 생성
```
# 마스터 컨테이너에 접속한 다음 SSH 키 생성
docker container exec -it master ssh-keygen -t rsa -C ""
```
* docker-compose.yml 파일 수정
```yaml
version: "3"
services:
  master:
    container_name: master
    image: jenkinsci/jenkins:2.142-slim
    ports:
      - 8080:8080
    volumes:
      - ./jenkins_home:/var/jenkins_home
# links 요소를 사용해 다른 sevice 그룹 컨테이너와 통신한다.
    links:
      - slave01
  slave01:
    container_name: slave01
    image: jenkinsci/ssh-slave
    # master 컨테이너에서 만든 ssh public key를 적어준다.
    environment:
      - JENKINS_SLAVE_SSH_PUBKEY=ssh-rsa ~~~~~~
```
* 컨테이너 실행 후 slave01 노드 추가
  * SSH 아이디 (jenkins) / 비밀키를 이용하여 인증정보를 추가한다.
  * Host Key Verification Strategy : Non verifying Verification Strategy



