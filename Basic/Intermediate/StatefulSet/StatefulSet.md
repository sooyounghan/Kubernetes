-----
### StatefulSet
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/d5fefbc6-5afc-4808-8a31-d1cf3982c905">  
</div>

1. Application 종류 : Stateless, Stateful Application
   - Stateless : Web Server (종류 : Apache, Nginx, IIS 서버 등)
   - Stateful : DataBase (종류 : MongoDB, MariaDB, Redis 등)

2. Stateless : App이 여러개 배포되더라도 동일한 서비스 역할을 함
   - Volume이 반드시 필요하지 않음 :  만약, App의 로그를 영구적으로 저장하고 싶으면 Volume과 연결할 수 있는데, Volume 하나의 모든 App이 연결이 되서 필요한 경우 로그 저장 가능
3. Stateful : App마다 자신의 역할이 존재
   - MongoDB의 경우 하나는 Primary 역할, 또 하나는 Secondary, 다른 하나는 Arbiter 역할
     + Primary : Main DB
     + Primary가 죽으면 Aribter가 이를 감지해 Secondary가 Primary 역할을 하도록 변경
   - 따라서, 만약 App 하나가 죽으면 Stateless App은 단순히 같은 서비스의 역할을 하는 App을 복제해주면 되지만, Stateful App은 Arbiter의 역할을 하는 App이 죽으면, 반드시 Arbiter의 역할을 하는 App이 만들어져야 되고, App 이름도 이름 자체가 고유 아이덴티티 식별 요소이므로 변경하면 안 됨
   - 역할이 다른 만큼, Volume도 각각 사용 : Arbiter와 같은 역할을 하는 App이 죽어도 새 App이 볼륨과 다시 연결이 되면서 해당 역할을 이어갈 수 있음

<div align="center">
<img src="https://github.com/user-attachments/assets/d67f66b2-9d8e-4cdc-8668-8291fe8d069c">
</div>

4. App들의 연결이 되는 대상과 네트워킹 흐름에 대한 차이
   - Stateless : 대체로 사용자들이 접속을 하고 이 접속된 네트워크 트래픽은 한 App에 집중되면 서버에 문제가 생기므로 그렇지 않도록 여러 App에 분산이 되는 형태로 흘러가게 됨
   - Stateful : 대체로 내부 시스템들이 데이터 베이스 저장을 위해서 연결하므로, 트래픽은 각 App에 맞게 들어와야 함
     + Primary App은 Read-Write 권한이 있으므로 연결을 하려는 시스템에서 CRUD를 모두 하려면 접속을 해야함
     + Secondary App은 Read 권한만 있으므로 조회만 해도 되는 내부 시스템은 App 2에 접속을 할 수 있음
     + App 3은 Primary와 Secondary 상태를 감시하고 있어야 하므로 App 1과 App 2에 연결이 되어야 함
   - 즉 Stateful App은 역할에 따라 의도가 있는 연결, Stateless App은 단순 분산 목적만 가지고 있는 연결
   - Kubernetes는 두 App의 특징에 따라 활용할 수 있도록 나눠짐
     + Stateless은 ReplicaSet Controller
     + Stateful은 StatefulSet Controller

<div align="center">
<img src="https://github.com/user-attachments/assets/9b87bf29-62fc-4dfe-8287-a69cd6933bd4">
</div>

5. StatefulSet Controller
   - Replica1 Controller를 만들게 되면 StatefulSet도 ReplicaSet과 마찬가지로 해당 개수만큼 Pod가 관리
   - 다른 점은 Pod가 생길 때, ReplicaSet은 뒷부분이 랜덤으로 생성되지만, StatefulSet은 0부터 순차적으로 숫자가 붙어서 생성
   - ReplicaSet을 3으로 늘리면, ReplicaSet은 2개의 추가적 Pod가 동시에 생성되고 이미 이름이 생성되는 것에 비해, StatefulSet은 2개를 늘리더라도 동시에 안 생기고 하나씩 순차적 생성
     + 이 상황에서 하나의 Pod가 삭제가 되면 ReplicaSet은 또 새로운 이름으로 Pod 생성
     + StatefulSet은 삭제된 기존 이름과 동일한 이름의 Pod가 재생성됨
     + ReplicaSet을 0으로 줄이면, ReplicaSet은 모든 Pod가 동시에 삭제되지만, StatefulSet은 인덱스가 높은 Pod부터 순차적 삭제
    
   - Headless와 PVC
     + ReplicaSet은 Pod의 Volume을 연결하려면 PVC를 별도로 직접 생성해놔야 하며, ReplicaSet을 만들면 템플릿에 Pod의 내용에 persistentVolumeClaim으로 pvc1을 지정해놓으므로 바로 연결
     + StatefulSet의 경우, 템플릿을 통해 Pod가 만들어지고 추가적으로 volumeClaimTemplate를 통해 PVC가 동적 생성이 되고 Pod와도 바로 연결
     + ReplicaSet을 3으로 변경하면, 모든 Pod들이 동일한 PVC 이름으로 PVC1을 가지고 있으므로, 한 PVC에 연결
     + Stateful의 경우에는 volumeClaimTemplate이므로 Pod가 추가될 때마다 새로운 PVC가 만들어지면서 연결이 되어, Pod마다 각자의 역할에 따른 데이터 저장 가능
       + Pod2가 삭제가 되면, Pod2가 그대로 만들어지면서 기존의 Pod2의 연결이 되어있던 PVC에 붙게 됨
       + replicas를 0으로 줄이면, StatefulSet은 Pod를 순차적으로 삭제하지만 PVC는 삭제하지 않음 : Volume은 함부로 지우면 안 되므로 사용자가 직접 삭제해야 함

   - ServiceName
     + 이 이름과 매칭되는 Headless 서비스를 만들게 되면 Pod에 예측 가능한 도메인 이름이 만들어지므로, Internal Server의 특정 Pod 입장에서 원하는 StatefulSet의 Pod에 연결 가능
     + 따라서 상황에 따라 Pod를 선택해 접속 가능
