

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
  - 쿼버네티스 클러스터 내에서만 IP를 통해 파드로 접근할 수 있다.
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
  - nodeSelector
    - 파드를 만들 때 nodeSelector 항목에 원하는 노드의 키/밸류를 써주면 된다.
    - 파드를 만들때 resources 항목에 필요한 메모리, 메모리 리밋 등을 넣어 줄 수 있다.



### Service - ClusterIP, NodePort, LoadBalancer

- [강의자료](https://kubetm.github.io/practice/beginner/object-service/)

- ClusterIP, NodePort, LoadBalancer 서비스의 타입 이름이다.



- ClusterIP 타입 서비스
  - 기본 타입
  - 클러스터 안의 노드나 파드에서 ClusterIP 타입 서비스를 이용해서 서비스에 연결된 파드에 접근한다.
  - 파드에도 IP가 있어서 그 IP로 파드에 접근 할 수 있지만 그 IP는 문제가 생기면 다시 할당되기 때문에 신뢰성이 떨어진다.
  - ClusterIP 타입 서비스의 클러스터 IP에 접근하면 항상 연결되어 있는 파드에 접근할 수 있다.
  - ClusterIP 타입 서비스는 삭제되거나 재생성 되지 않는다.
  - 어떤 상황에 쓰나?
    - 인가된 사용자 (운영자)
    - 내부 대쉬보드
    - Pod의 서비스상태 디버깅



- NodePort 타입 서비스

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

emptyDir

Pod 안에 있는 Volume

Pod 날라가면 날라간다.



hostPath

이름 그대로 파드가 올라가 있는 노드의 패스를 Volume으로 사용

단점으로 파드가 다른 노드에 올라가면 사람이 추가적인 작업을 해줘야한다.

노드에 있는 데이터를 파드에서 쓰기위한 용도

파드에 데이터를 저장하기 위한 용도가 아니다는 점



PV/PVC

볼륨은 로컬일 수도 있고 클라우드에 연결 할 수도 있다.

이런 각각의 Persistent Volume과 연결

Pod는 Persistent Volume에 직접 연결하는게 아니라 Persistent Volume Claim을 통해서 PV로 붙는다.

User 영역 Pod -> PVC

Admin 영역 PV, Volume





### ConfigMap, Secret - Env, Mount

- [강의자료](https://kubetm.github.io/practice/beginner/object-configmap_secret/)

특정 정보 때문에 Dev와 Prod 이미지를 따로 가져갈 수는 없다.

SSH, User 같은 상수 값 모아서 ConfigMap

Key 값은 Secret



### Namespace, ResourceQuota, LimitRange

- [강의자료](https://kubetm.github.io/practice/beginner/object-namespace_resourcequota_limitrange/)









## [기초편] 컨트롤러



### Replication Controller, ReplicaSet - Template, Replicas, Selector

Replication Controller - Deprecated

ReplicaSet - Replaced

Selector라는 기능은 ReplicaSet에만 있다.



**Controller 기능**

- Auto Healing
  - 파드나 파드가 있는 노드가 내려가면 즉각적으로 인지하고 새로운 노드에 파드를 생성해 준다.

- Auto Scaling
  - 파드에 리소스의 리미트 상태가 되면 파드를 하나 더 만들어서 부하를 분산 시켜서 파드가 죽지 않게 않다.

- Software Update
  - 여러 파드에 버전을 업그레이드 하거나 롤백하는 기능 제공

- Job
  - 컨트롤러가 필요한 순간에만 파드를 만들어서 해당 작업을 이행하고 파드를 삭제



설정 파일 spec에 template, replicas, selector

Selector의 matchLabels, matchExpression



라벨로 Replication Controller 오브젝트와 파드를 연결

Replication Controller는 template에 있는 내용을 바탕으로 파드를 생성 하거나 업데이트 한다.



replicas에 적힌 숫자 만큼 파드를 만들어준다.

Scale Out, Scale In 가능

생성된 파드가 없는 상태에서 replicas 2를 주면

컨트롤러 오브젝트가 template으로 파드 2개를 생성해 준다.



selector에 matchLabels는 Replication Controller와 마찮가지로 라벨에 키벨류가 같아야 연결

matchExpression 키와 벨류를 디테일하게 컨트롤

**matchExpression 옵션**

Exists

key가 A 이고 operator: Exists

벨류가 달라도 키 값이 A인 파드를 다 선택



DoesNotExist

key: A 이고 operator: DoesNotExist

키에 A가 들어가지 않은 파드를 선택



In

key: A 이고 operator: In

키에 A인 파드들 중에 밸류가 2와 3인 파드를 선택



NotIn

key: A 이고 operator: NotIn

키에 A인 파드들 중에 밸류가 2와 3이 아닌 파드를 선택



### Deployment - Recreate, RollingUpdate







### DaemonSet, Job, CronJob





## [중급편] Pod



## [중급편] 기본 오브젝트



## [중급편] 컨트롤러











