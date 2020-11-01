

## [기초편] 기본 오브젝트



### Pod - Container, Label, NodeSchedule

- [강의자료](https://kubetm.github.io/practice/beginner/object-pod/)

- 클러스터, 마스터, 노드, 파드, 컨테이너 

- 쿠버네티스 클러스터는 크게 두 종류의 서버로 구성한다.
  - 클러스터를 관리하는 마스터와 실제 컨테이너를 실행시키는 노드
- 파드는 여러 노드들 중에 한 노드에 올라가야 한다.
  - 파드 안에 각각의 컨테이너는 하나의 독립적인 서비스를 구동할 수 있다.
    - 즉 파드 안에서 여러 앱들이 돌아갈 수 있다.
  - pod는 배포 단위



- Container
  - 하나의 파드 안에서 컨테이터 끼리는 localhost:상대방 포트번호
    - 하나의 파드 안에 컨테이너들 끼리 포트가 중복되면 안된다.
  - 생설 될 때 파드에 IP가 자동으로 할당 되는데 재생성시 변경된다.
  - 쿠버네티스 클러스터 내에서만 IP를 통해 파드로 접근할 수 있다.
    - 외부에서는 IP 주소로 파드로 접근할 수 없다.



- Label
  - Pod에 라벨은 여러개 설정할 수 있다.
  - 원하는 라벨만 **서비스에 연결**해서 사용자에게 제공하면 된다.
  - 강의자료 예제
    - type:web
      - 키 type 벨류 web
    - lo:dev
      - 키 lo 벨류 dev



- NodeSchedule
  - 파드는 결국 여러 노드들 중에 한곳에 올라가야는데 2가지 방법이 있다.
    - 내가 직접 선택
      - nodeSelector
        - 파드를 만들 때 nodeSelector 항목에 원하는 노드의 키/밸류를 써주면 된다.
    - 스케줄러가 판단



### Service - ClusterIP, NodePort, LoadBalancer

- [강의자료](https://kubetm.github.io/practice/beginner/object-service/)

- ClusterIP, NodePort, LoadBalancer 서비스의 타입 이름이다.



- ClusterIP 타입 서비스
  - 기본 타입
  - 다른 노드에서도 가능
  - 클러스터 안의 노드나 파드에서 ClusterIP 타입 서비스를 이용해서 서비스에 연결된 파드에 접근한다.
  - 파드에도 IP가 있어서 그 IP로 파드에 접근 할 수 있지만 그 IP는 문제가 생기면 다시 할당되기 때문에 신뢰성이 떨어진다.
  - ClusterIP 타입 서비스의 클러스터 IP에 접근하면 항상 연결되어 있는 파드에 접근할 수 있다.
  - ClusterIP 타입 서비스는 삭제되거나 재생성 되지 않는다.
  - 어떤 상황에 쓰나?
    - 인가된 사용자 (운영자)
    - 내부 대쉬보드
    - Pod의 서비스상태 디버깅



- NodePort 타입 서비스

  - External에서 노드 IP 서비스가 할당한 포트로 해서 접속 할 수 있다.
  - 내부에서는 ClusterIP와 같이 서비스IP로 붙으면 파드에 접근 할 수 있다.
  - 클러스터에 연결되어 있는 모든 노드에 똑같은 포트를 할당
  - 서비스에 지정된 포트 번호만 사용하면 파드에 접근할 수 있다.
  - 노드의 IP와 서비스가 할당한 포트 번호로 노드에 접근하면 서비스에 연결된 파드들에 트레픽이 분산 되어서 간다. 
    - 강의 자료에 그림 참고
    - 어떤 노드의 IP로 접근을 하더라도 노드들의 파드에 트래픽이 분산 되어서 전달된다. 
    - 특정 노드 위에 파드에게만 트래픽을 전달 할 수도 있다.
      - externalTrafficPolicy: Local

  - 어떤 상황에 쓰나?
    - 클러스터 밖에는 있지만 내부망 안에서 접근을 할 때 쓰인다.
    - 데모나 임시 연결용



- LoadBalancer 타입 서비스 
  - NodePort 타입 서비스 + 로드밸런서
  - 클라우드에서 제공하는 로드밸런서와 파드를 연결한 후 해당 로드밸러서의 IP를 이용해 클러스터 외부에서 파드에 접근할 수 있게 해준다.
  - LoadBalancer 타입 서비스는 로드밸런서 IP와 할당한 포트 번호로 외부에서 붙고
    - NodePort 타입 서비스는 노드의 IP와 할당한 포트 번호로 외부에서 붙고
  - 클라우드 서비스가 아니면 외부 IP를 할당하는 IP가 필요한데 없으면 서비스 생성이 pending 된다.
  - 어떤 상황에 쓰나?
    - 외부 시스템 노출용



### Volume - emptyDir, hostPath, PV/PVC

- [강의자료](https://kubetm.github.io/practice/beginner/object-volume/)

- emptyDir
  - Pod 안에 있는 Volume
  - Pod 생성시 만들어지고 삭제시 없어짐
  - Pod 날라가면 날라간다.
  - 이 Volume에 쓰이는 데이터는 일시적으로 사용할 목적의 데이터



- hostPath
  - 이름 그대로 파드가 올라가 있는 노드의 패스를 Volume으로 사용
  - 단점으로 파드가 다른 노드에 올라가면 사람이 추가적인 작업을 해줘야한다.
  - 노드에 있는 데이터를 파드에서 쓰기위한 용도
  - 파드에 데이터를 저장하기 위한 용도가 아니다는 점
  - 사전에 해당 Node에 경로가 있어야한다.



- PV/PVC
  - 볼륨은 로컬일 수도 있고 클라우드에 연결 할 수도 있다.
    - 이런 각각의 볼륨을 Persistent Volume과 연결
  - Pod는 Persistent Volume에 직접 연결하는게 아니라 Persistent Volume Claim을 통해서 PV로 붙는다.
  - User 영역 Pod -> PVC
  - Admin 영역 PV, Volume
  - 흐름
    - PV 정의 생성
    - PVC 생성
    - PV 연결
    - Pod 생성시 PVC 마운팅
  - 1대1
  - 1대 다
  - 클레임 요구한다 충분한 용량을



### ConfigMap, Secret - Env, Mount

- [강의자료](https://kubetm.github.io/practice/beginner/object-configmap_secret/)

- 특정 정보 때문에 Dev와 Prod 이미지를 따로 가져갈 수는 없다.

- 상수 값 모아서 ConfigMap으로
  - 키 밸류, 키 밸류

- 보안적인 관리가 필요한 값을 모아서 Secret으로
  - 키 밸류, 키 밸류
    - 밸류는 Base64 인코딩을 해서 넣어줘야 한다.
      - 파드로 들어갈 때는 디코딩이 되서 환경변수에서는 원래의 값이 보이게 된다.
    - 밸류의 보안적인 요소는 인코딩해서 값을 가지고 있는게 아니다.
      - 값이 디비에 있지 않고 메모리에 있다는 점이 보안적인 요소라네.
        - 1 Mbyte
        - 많이 만들면 성능에 문제가 생길 수도 있다고



- Env (Literal)
  - ConfigMap, Secret 키 밸류 상수 정의하는 방법 



- Env (File)
  - 파일을 ConfigMap으로
    - 파일 이름이 키 내용이 밸류
      - 파일 이름을 키로 쓰지 않고 키 따로 정의
      - 설정 파일 참고
    - Secret도 동일함.
      - Secret은 만들어질 때 Base64로 인코딩을 한다.
        - 파일 내용이 이미 인코딩 되어 있으면 두번 인코딩 되니깐 주의



- Volume Mount (File)
  - Env(File)에 마운트 패스가 추가된다.
  - 마운트는 원본과 연결 시켜놓는다는 개념이라네.
    - ConfigMap이 변하면 Pod의 내용도 변한다.
    - Env(File)는 한번 주입해주면 끝.
    - 장치를 마운트한다.
      - 장치를 특정 파일/폴더에 할당한다는 뜻



### Namespace, ResourceQuota, LimitRange

- [강의자료](https://kubetm.github.io/practice/beginner/object-namespace_resourcequota_limitrange/)



- Namespace
  - Namespace 내에서 이름 중복 안된다.
  - Namespace 별로 분리해서 관리된다.
    - PV, Node 등 Namespace들이 공용으로 사용하는 오브젝트도 있다.
  - Namespace를 지우면 그 안에 있는 자원들도 모두 지워지니깐 유의해야 한다.



- ResourceQuota
  - Pod들이 이 조건에 맞춰서 나눠쓴다.
  - 이름 그대로 쿼터임.
    - cpu, memory, storage, 오브젝트의 숫자 등
  - requests
  - limits



- LimitRange
  - 이것도 이름 그대로 레인지
  - 하나의 Pod에 대한 설정
  - min
  - max
  - maxLimitRequestRatio



## [기초편] 컨트롤러



### Replication Controller, ReplicaSet - Template, Replicas, Selector

- [강의자료](https://kubetm.github.io/practice/beginner/controller-replicationcontroller_replicaset/)

- Replication Controller - Deprecated

- ReplicaSet - Replaced

- Selector라는 기능은 ReplicaSet에만 있다.



- Controller 기능
  - Auto Healing
    - 파드나 파드가 있는 노드가 내려가면 즉각적으로 인지하고 새로운 노드에 파드를 생성해 준다.
  - Auto Scaling
    - 파드에 리소스의 리미트 상태가 되면 파드를 하나 더 만들어서 부하를 분산 시켜서 파드가 죽지 않게 않다.
  - Software Update
    - 여러 파드에 버전을 업그레이드 하거나 롤백하는 기능 제공
  - Job
    - 컨트롤러가 필요한 순간에만 파드를 만들어서 해당 작업을 이행하고 파드를 삭제



- Template
  - Controller는 파드가 죽으면 Template을 가지고 파드를 만들어 준다.
  - 라벨로 Replication Controller 오브젝트와 파드를 연결
  - Replication Controller는 template에 있는 내용을 바탕으로 파드를 생성 하거나 업데이트 한다.



- Replicas
  - replicas에 적힌 숫자 만큼 파드를 만들어준다.
  - Scale Out, Scale In 가능
  - 생성된 파드가 없는 상태에서 replicas 2를 주면
    - 컨트롤러 오브젝트가 template으로 파드 2개를 생성해 준다.



- Selector
  - matchExpression 키와 벨류를 디테일하게 컨트롤
  - matchExpression 옵션
    - Exists
      - key가 A 이고 operator: Exists
      - 벨류가 달라도 키 값이 A인 파드를 다 선택
    - DoesNotExist
      - key: A 이고 operator: DoesNotExist
      - 키에 A가 들어가지 않은 파드를 선택
    - In
      - key: A 이고 operator: In
      - 키에 A인 파드들 중에 밸류가 2와 3인 파드를 선택
    - NotIn
      - key: A 이고 operator: NotIn
      - 키에 A인 파드들 중에 밸류가 2와 3이 아닌 파드를 선택



### Deployment - Recreate, RollingUpdate

- [강의자료](https://kubetm.github.io/practice/beginner/controller-deployment/)
- 서비스 운영중에 재배포 할 때 도움을 주는 컨트롤러

- 배포 방법
  - ReCreate
  - Rolling Update
  - Blue/Green
  - Canary
    - Ingress Controller



- Recreate
  - Deployment 타입 Recreate
  - Deployment가 ReplicaSet 생성
  - revisionHistoryLimit
    - 이름 그대로 revisionHistoryLimit를 1로 두면 새로운 ReplicaSet이 만들어 질 때 앞에 ReplicaSet을 지우지 않느다.
      - 기본은 10개를 남긴다고



- RollingUpdate
  - Deployment 타입 RollingUpdate
  - ReplicaSet이 추가로 생성 되면서 점진적으로 업데이트 된 파드를 늘려간다.





### DaemonSet, Job, CronJob

- [강의자료](https://kubetm.github.io/practice/beginner/controller-daemonset_job_cronjob/)



- DaemonSet
  - 노드의 자원 상태와 상관없이 노드에 파드가 하나씩 생긴다는 특징이 있다.
    - ReplicaSet은 자원을 봐가면서 Pod가 생길 수도 안 생길 수도 있다.
  - 한 노드에 하나 이상의 Pod를 만들 수는 없지만 Pod를 안만들 수는 있다.
  - 대표적인 3가지 예
    - Performance
    - Logging
    - Storage



- Job
  - 파드를 만들고 일이 끝나면 없앤다.
  - 파드를 만드는 주체에 따라서 상황별로 파드의 동작이 다르다.
    - 직접 만든 Pod
    - ReplicaSet을 통해서 만들어진 Pod
    - Job을 통해서 만들어진 Pod



- CronJob
  - Job을 만들고 잡들이 Pod를 만들고
  - Allow
  - Forbid
  - Replace





## [중급편] Pod



### Pod - Lifecycle

- Phase
  - Pod의 전체 상태를 대표하는 속성
- Conditions
  - Pod 생성되면서 실행하는 단계와 그 단계의 상태를 나타낸다.
- ContainerStatuses 안에 status
  - 컨테이너의 상태를 나타낸다.



### Pod - ReadinessProbe, LivenessProbe

- [강의자료](https://kubetm.github.io/practice/intermediate/pod-probe/)



- ReadinessProbe
  - App 구동 순간에 트래픽 실패를 없앤다.
- LivenessProbe
  - App 장애시 지속적인 트래픽 실패를 없앤다.



### Qos Classes

- Guaranteed
- Burstable
- BestEffort



### Pod - Node Scheduling

- [강의자료](https://kubetm.github.io/practice/intermediate/pod-node_scheduling/)



- 노드셀렉터
  - 노드를 선택하는 기능
  - 파드가 클러스터 안 어떤 노드에서 실행될지를 키-값 쌍으로 설정
  - 가장 간단한 스케줄링 옵션
  - 파드의 .spec 필드에 설정할 수 있는 노드셀렉터
- 어피니티와 안티 어피니티
  - 노드 어피니티
    - 노드셀렉터와 비슷하게 노드의 레이블 기반으로 파드를 스케줄링한다.
    - 노드 어피니티 두 가지 필드
      - requiredDuringSchedulingIgnoredDuringExecution
      - preferredDuringSchedulingIgnoredDuringExecution
  - 파드의 어피니티와 안티 어피니티
    - 파드들을 함께 묶어서 같은 노드에서 실행하도록 설정하는 어피니티
    - 파드들을 다른 노드에 나누어서 실행하도록 설정하는 안티 어피니티
      - CPU나 네트워크 같은 하드웨어 자원을 많이 사용하는 앱 컨테이너가 있을 때 여러 노드로 파드를 분산
- 테인트와 톨러레이션 사용하기
  - 쿠버테니스 클러스터의 특정 노드에 테인트를 설정할 수 있다.
    - 테인트를 설정한 노드에는 파드들을 스케줄링하지 않는다.
    - 테인트를 설정한 노드에 파드들을 스케줄링하려면 톨러레이션을 설정해야 한다.
      - 테인트는 톨러레이션에서 설정한 특정 파드들만 실행하고 다른 파드는 실행하지 못하게 한다.
  - 테인트와 톨러레이션의 하위 필드



## [중급편] 기본 오브젝트



## [중급편] 컨트롤러











