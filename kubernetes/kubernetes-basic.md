>[Slipp 스터디](https://www.slipp.net/wiki/pages/viewpage.action?pageId=41582977)에서 Docker&Kubernetes 수업을 하며 정리  
참고한 책 : [도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문](http://aladin.co.kr/shop/wproduct.aspx?ItemId=186771560)

## 1. 쿠버네티스

### 1.1 쿠버네티스란?

여러 서버에 걸쳐있는 컨테이너를 효율적으로 분산/배치/관리하는 기법으로 유용한 기능들을 제공
(컨테이너 배포, 스케줄링, 모니터링, 장애시 리커버리, 삭제 등)
작은 컨테이너라면 직접 배포하여 사용하면 되지만 컨테이너 수가 많아지며 해당 컨테이너를 어디에 배포해야 될지에 대한 결정이 필요해짐
* CPU, memory resource 고려해야하는 경우
* 동일 물리서버에 배포해야하는 경우
* 가용성을 위해 다른 물리서버에 배포해야 하는 경우

### 1.2 Object

쿠버네티스는 Basic Object와 이를 생성하고 관리하는 추가적인 기능을 가진 Controller로 이루어 진다. 이러한 오브젝트의 스펙(설정)이외에 추가적인 메타 정보들로 구성된다.

* Basic Object :  k8s에서 배포 및 관리되는 기본적인 구성단위 (기본 오브젝트 만으로 설정, 배포 가능)

    * Pod : 컨테이너화된 애플리케이션
    * Service : 로드밸런서
    * NameSpace : 패키지명 
    * Volume : 디스크


* Controller : 기본 오브젝트를 생성하고 관리하는 역할 (편리성)
    * ReplicaSet : Pod 관리 (replica 수 만큼 pod를 배포, 관리) 

    * Deployment : ReplicaSet 추상화된 개념. 실제 운영에서 ReplicatSet보다 Deployment를 사용 
    * StatefulSet  : 상태를 가지는 애플리케이션 운용 시 적합
    * Job, CronJob : 하나 이상의 pod를 생성해 지정된 pod가 정상 종료될 때까지 관리하는 리소스 (batch)
    * ... 

* Volume : PersistentVolume, PersistentVolumeClaim, StorageClass

### 1.3 쿠버네티스 명령어

* 기본 명령어

``` sh
# 매니페스트 배포
kubectl apply -f [매티페스트 파일]
 
# 매니페스트 삭제
kubectl delete -f  [매니페스트 파일]
 
# 클러스터 상태 확인
kubectl get pod[,replicaset,deployment]

# pod echo 컨테이너 logs 확인
kubectl logs -f simple-echo -c echo
```

## 2. 쿠버네티스의 리소스

### 2.1 파드(Pod)

<img src="/images/kubernetes/kube-pod-1.png">

* 기본적인 배포 단위(컨테이너가 모인 집합체)

* 쿠버네티스는 하나의 컨테이너를 개별적으로 배포하지 않고, pod 단위로 배포한다

* 결합(정합도, 결합도)이 강한 컨테이너를 pod로 묶어 일괄 배포한다 ex) web server + was 

* 같은 pod를  한 node에 여러개 배치가 가능하고, * 여러 노드에 배치가 가능하지만 한 pod가 2개이상의 node에 배치될 수 없다. 

<img src="/images/kubernetes/kube-pod-2.png">

* Pod 내의 컨테이너는 IP, port를 공유한다 (localhost를 통해 통신 가능)

* Pod 내의 배포된 컨테이너간에는 디스크 볼륨을 공유할 수 있다.

* 파드에는 고유한 가상 IP 주소가 할당되고 이 주소를 컨테이너가 공유하므로 컨테이너간 통신이 가능하다. 

* 이 가상 IP로 다른 파드와의 통신도 가능하다.

> Volume  
> Pod가 기동할때 기본적으로 컨테이너마다 로컬디스크를 생성한다.  
> 컨테이너가 리스타트 되거나 새로 배포될 때 로컬 디스크는 새롭게 정의되서 배포되므로 내용이 유실되기 때문에 영구적으로 저장할수 있는 스토리지 볼륨을 사용한다.  
> 쿠버네티스 볼륨은 Pod내의 컨테이너간 공유 가능하다.

### 2.2 레플리카세트(ReplicaSet)
* 동일한 pod를 여러개로 복제하여 생성/관리하는 리소스 (가용성 확보를 위해)

* 매니페스트 설정 파일 하나로 여러 파드를 관리할 수 있다.

* 파드명은 무작위 접미사가 붙으므로 삭제된 파드는 복원 불가
→ 때문에 stateless application에 적절하다 (webserver, was)

### 2.3 디플로이먼트(Deployment)
* 레플리카세트를 관리하고 다루기 위한 상위 리소스로 애플리케이션 배포(deploy)의 단위

* 디플로이먼트는 레플리카세트를 제어하여 파드의 개수 증감, 버전 교체가 가능하며, 추가로 이전 버전으로 롤백등 기능을 가진 리소스

* 레플리카세트의 리비전 관리 가능 (–record)

* 실제 운영에서는 레플리카세트를 직접 다르기 보다는 디플로이먼트 매니페스트 파일을 통해 운영

<img src="/images/kubernetes/kube-deployment-1.png">

### 2.4 서비스(Service)
* 랜덤으로 IP주소가 생성되는 Pod에 고정적인 엔드포인트(ip)로 호출할 수 있는 기능을 제공하는 리소스.

* 여러 pod에 같은 애플리케이션 운용 시 pod간의 로드밸런싱이 필요한데 이를 서비스가 해결한다.

* 파드의 집합(replicaset)에 대한 경로, 주소가 동적으로 바뀌더라도 클라이언트가 접속대상을 바꾸지 않고 하나의 이름으로 접근하도록 하는 기능을 제공(DNS)

* 멀티 포트 지원 : 서비스는 하나의 포트가 아닌 여러개의 포트를 동시에 지원 할 수 있다. ex) HTTP, HTTPS

<img src="/images/kubernetes/kube-service-1.png">

#### 2.4.1 ClusterIP 서비스 
쿠버네티스 클러스 내부 IP주소에 서비스를 공개하여, 쿠버네티스 클러스터 내에서는 서비스 접근이 가능하나 외부 IP를 할당 받지 못해 외부로부터 접속이 불가능하다.

#### 2.4.2 NodePort 서비스 (L4)
* 클러스터 외부에서 접속할 수 있는 서비스
* ClusterIP를 만들지만 각 노드에서 서비스 포트로 접속하기 위한 글로벌 포트를 개방한다는 차이점이 있음

#### 2.4.3 LoadBalancer 서비스
* 일반적으로 클라우드 벤더에서 제공하는 설정 방식으로 플랫폼에서 제공하는 로드밸런서와 연동하기 위해 사용한다.

* 외부 IP를 가지는 로드밸런서를 할당하여 외부 접근 가능하다.
(GCP Cloud Load Balancing, AWS Elastic Load Balancing 지원)

#### 2.4.4 ExternalName 서비스
* 쿠버네스트 클러스터에서 외부 호스트를 네임 레졸루션 하기 위한 별명을 제공한다.(셀렉터/포트정의 없음)
* 외부 서비스를 쿠버네티스 내부에서 호출하고자 할 때 사용할 수 있다. (RDS, CloudSQL등 사용 시 이용)

### 2.5 인그레스(Ingress)
* 외부로 서비스 공개 시 서비스를 NodePort로 노출 하여 L4에 TCP단에서 pod를 밸런싱한다. 
HTTP/HTTPS 경로 기반으로 서비스 제어 필요 시 L7 레벨의 Ingress 사용하여 해결 가능 ( + 로드밸런싱, 인증서 처리등 가능)
<img src="/images/kubernetes/kube-ingress-1.png">

> Ingress의 구현체  
> - GKE : 글로벌 로드밸런서  
> - NginX ingress controller  
> - Kong ingress controller  

### 2.6 잡(Job)
* 하나 이상의 pod를 생성해 지정된 pod가 정상 종료될 때까지 관리하는 리소스

* 배치 작업 위주의 애플리케이션에 적합  

* Job에서 관리되는 pod는 job 종료 시 pod 같이 종료한다. 

* Job이 생성한 pod는 정상 종료 후에도 삭제되지 않고 남아있어 pod의 로그나 실행 결과를 분석 할 수 있다.

> Job의 특징  
> - completion : 데이터가 커 범위를 나눠 작업해야하는 경우 pod를 순차적으로 여러번 실행할 수 있다.   
> - parallelism : 여러 작업 처리 중 순차적 처리가 필요없는 경우 병렬처리가 가능하다.  
> - restartPolicy : Job 결과가 실패 일 경우 설정에 따라 재실행 할 지 설정할 수 있다.  
>   - always : Pod가 종료하면 항상  재실행하여 실행 상태 유지 (pod는 이 값이 기본)  
>   - Never : 실패 시 pod를 재생성해 실행  
>   - OnFailure : 실패 시 실패한 pod를 재실행  
>   - restartPolicy : 파드 종료 후 재실행 여부로 Job 리소스 Always로 설정불가능(Never 혹은 OnFailure 설정)  



### 2.6 크론잡(CronJob)
* 배치성 작업은 주기적으로 자동화하여 실행이 필요하다.
* Job은 단 한번만 실행되는데 반해 CronJob 스케줄을 지정해 정기적으로 pod를 실행할 수 있다.
컨테이너에 친화적인 특성을 유지하며 이벤트 애플리케이션을 따로 사용하지 않으며 스케줄에 따른 작업을 수행할 수 있다.
* 컨테이너에 작업을 매니페스트 파일로 기술하거나 구현을 매니페스트 파일 대신 도커 이미지에 포함시키는 방법 사용