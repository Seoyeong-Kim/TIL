# Docker Study
>[Slipp 스터디](https://www.slipp.net/wiki/pages/viewpage.action?pageId=41582977)에서 Docker&Kubernetes 수업을 하며 정리  
>참고한 책 : [도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문](http://aladin.co.kr/shop/wproduct.aspx?ItemId=186771560)

## 1. Docker
### 1.1. 컨테이너형 가상화
* 컨테이너형 가상화 기술로 가상화 소프트웨어 없이 운영체제의 리소스를 격리해 가상 운영체제를 만들 수 있다.
* 호스트형 가상화 기술은 호스트 운영체제를 거쳐 하드웨어를 제어하기 때문에 그만큼 오버헤드가 발생한다.
* 컨테이너에 애플리케이션이 함께 배포되며 간단하게 환경의 영향을 덜 받고 배포 할 수 있다.
* 각 컨테이너는 별도의 OS를 가지지 않고 Docker Engine을 공유한다.
<img src="/images/docker/docker-basic-1.png" width="40%">

### 1.2. Docker Engine
* Client와 Server로 나누어져 있고 REST API로 소통을 한다.
* 클라이언트는 요청만 하고 실질적인 `build`, `run`, `push` 등의 작업은 서버가 수행한다.
<img src="/images/docker/docker-basic-2.png" width="50%">
 
## 2. 실습
### 2.1. 도커 컨테이너 실행하기
* helloworld 파일 작성
	```bash
	#!/bin/sh
	echo "Hello, World!"
	```
* Dockerfile 작성
	```bash
	# FROM 절은 컨테이너의 원형 역할을 할 도커 이미지(운영 체제)를 정의한다.
	FROM ubuntu:16.04
	
	# COPY 절은 셸 스크립트 파일(helloworld)을 도커 컨테이너 안의 /usr/local/bin에 복사하라고 정의한 것이다.
	COPY helloworld /usr/local/bin
	# RUN 절은 도커 컨테이너 안에서 어떤 명령을 수행하기 위한 것이다.
	RUN chmod +x /usr/local/bin/helloworld
	
	# CMD 절은 완성된 이미지를 도커 컨테이너로 실행하기 전에 먼저 실행할 명령을 정의한다.(명령을 공백으로 나눈 배열로 나타냄)
	CMD ["helloworld"]
	```
* 도커 이미지 빌드, 컨테이너 실행
	```
	docker image build -t helloworld:latest
	docker container run helloworld:latest
	```
### 2.2 간단한 애플리케이션과 도커 실행하기
> go 언어로 만든 간단한 웹서버를 도커 컨테이너에서 실행
* main.go 작성 (8080포트로 오는 모든 요청에대해 Hello Docker 응답을 보냄)
	```go
	package main

	import (
		"fmt"
		"log"
		"net/http"
	)

	func main() {
		http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
			log.Println("received request")
			fmt.Fprintf(w, "Hello Docker!!")
		})

		log.Println("start server")
		server := &http.Server{Addr: ":8080"}
		if err := server.ListenAndServe(); err != nil {
			log.Println(err)
		}
	}
	```  
* Dockerfile 작성
	```dockerfile
	FROM golang:1.9

	RUN mkdir /echo
	COPY main.go /echo

	CMD ["go", "run", "/echo/main.go"]
	```
* 도커 이미지 빌드
	```bash
	docker image build -t example/echo:latest .
	docker image ls
	```
* 컨테이너 실행
	```bash
	docker container run example/echo:latest
	docker container run -d example/echo:latest
	```
* 애플리케이션에 요청 보내기
	```bash
	curl http://localhost:8080/
	```
* 도커 컨테이너 종료
	```bash
	docker container stop $(docker container ls --filter "ancestor=example/echo" -q)
	docker container run -d -p 9000:8080 example/echo:latest
	```