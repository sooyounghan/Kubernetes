-----
### Kubernetes Overview
-----

<img width="1566" height="753" alt="image" src="https://github.com/user-attachments/assets/c36ac535-2af4-49fc-a5a8-6a2a050a0e85" />

1. 서버 한 대는 Master, 다른 서버는 Node라고 해서 한 마스터에 여러 노드들이 연결
   - 연결이 되면 하나의 Kubernetest 클러스터라는 개념으로 묶임
   - Master : Kubernetes의 전반적 기능들을 컨트롤하는 역할
   - Node : 자원을 제공하는 역할

2. Cluster 안에 Namespace라는 객체가 Kubernetes Object들을 독립된 공간으로 분리하게 만들어줌
   - Pod : 최소 배포 단위인 Pod들이 있고, 이 Pod들에게 외부로부터 연결이 가능하도록 IP를 할당해주는 서비스가 존재
     + 서로 다른 Namespace에 있는 Pod들에게 연결을 할 수 없음
     + Pod 안에는 여러 Container가 존재 : 컨테이너 하나 당 하나의 App이 동작하므로 결국 Pod에는 여러 App이 가동될 수 있음
   - 하지만, Pod에 문제가 생겨서 재생성이 되면, 그 안의 데이터들은 모두 사라짐
     + Volume : 따라서, 볼륨(Volume)을 만들어서 Pod에 연결하면 데이터는 이 볼륨에 저장할 수 있음
     + 즉, Pod가 재생성되서 데이터가 날아가는 문제 해결 가능
   - ResourceQuota와 LimtRange : Namespace 내 ResourceQuota와 LimtRange를 사용하여, Namespace에서 사용할 자원의 양 한정 가능 : Pod 개수 제한 또는 CPU나 메모리 제한 가능
   - ConfigMap와 Secret : Pod 생성 시에, 컨테이너 안에 환경 변수 값을 넣어주거나 파일을 마운팅을 해줄 수 있는데, ConfigMap / Secret을 통해 설정 가능
   - Controller : Pod들을 관리하는 역할로, 종류가 다양
     + Replication Contorller, Replica Set : 가장 기본적 컨트롤러로, Pod가 죽으면 이를 감지해서 다시 살려주거나 Pod의 개수 조절 가능 (Scaling 기능)
     + Deployment : 배포 후에 Pod들을 새 버전으로 업그레이드해주며, 업그레이드 중 문제가 생기면 Rollback을 다시 쉽게 하도록 도와줌
     + DaemonSt : 한 Node의 Pod가 하나씩만 유지가 되도록 해줌
     + Job : 어떤 특정 작업만 하고 종료를 시켜야 하는 일을 할 때 Pod가 동작하도록 설정 (주기적으로 실행해야 할 때는 CronJob 사용)
