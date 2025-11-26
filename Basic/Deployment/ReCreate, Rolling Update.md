-----
### ReCreate, Rolling Update
-----
<img width="1538" height="729" alt="image" src="https://github.com/user-attachments/assets/e1ec411b-1a5f-408e-8eec-7c4e7dfc5888" />

1. Deploy
   - Deploy를 만들 때, selector, replicas, template 값을 똑같이 넣지만, 이 값들은 직접 이 Deployment가 Pod를 만들어서 관리하기 위한 값이 아니고, ReplicaSet을 만들고, 값을 지정하기 위한 용도
   - 만들어진 ReplicaSet은 본인의 역할인 내용대로 Pod들을 만들게 됨
   - 그리고 Service를 만들어서 Label을 연결하면, 이 Service를 통해 Pod에 접근할 수 있게 됨

2. Recreate
   - 템플릿 버전을 V2으로 업데이트
   - 먼저 Deployment는 Replicas의 값을 0으로 변경 : Pod들이 제거되고, Service도 연결 대상이 없어지므로 Down-Time 발생
   - 새로운 ReplicaSet을 생성 : 이 Template에는 변경된 V2의 Pod를 넣으므로, Pod들이 V2 버전으로 생성
   - Label이 있으므로, Service는 자동적으로 Pod들이 연결
   - Yaml 파일
     + deployment를 생성할 때, selector와 replicas, template이 필요
     + 배포 방식 지정 : strategy.type: 에 Recreate 설정
     + revisionHistoryLimit : 새로운 ReplicaSet이 만들었지만, 기존 ReplicaSet은 지우지 않았는데, 이 상태에서 새로운 버전으로 업그레이드를 하게 되면 ReplicaSet도 지워지지 않고 남아있으므로, 새로운 ReplicaSet이 만들어짐
       * 1로 설정하면, 이 상태에서 새로 업그레이드를 할 때, 새로운 ReplicaSet이 생기면서 0이 되고, ReplicaSet은 삭제 (즉, 0인 ReplicaSet을 하나만 남기겠다는 의미)
       * 또한, 이 값은 Optional 값이므로 값을 아예 넣지 않으면 기본 10개를 남기게 됨
       * 즉, 이렇게 0이 된 ReplicaSet은 이전 버전으로 되돌아가고 싶을 때 사용

3. Rolling Update
   - 서비스가 운영 중인 상태에서, 새로운 버전으로 템플릿을 교체되면서 Rolling Update 시작
   - 먼저, ReplicaSet을 하나 만들고, ReplicaSet이 1이므로, Pod가 하나 만들어지면서, V1 Label과 똑같으 라벨이 만들어지면서 Service 연결
   - 이제 이 Service에 연결하면 V1과 V2의 트래픽이 분산되어 보내지게 됨
   - 그런 다음, ReplicaSet을 1로 변경하면서 Pod가 하나 줄어들고, 삭제가 완료되면 V2의 Replicas를 2로 만들어서 하나를 더 늘리고, 마지막으로 V1을 0으로 만들면서 남은 Pod를 없앰
   - ReplicaSet을 지우지는 않고 배포를 종료
   - Yaml 파일
     + strategy.type: rollingUpdate
     + minReadySeconds: 10 - 이 값 없이 업데이트를 하게 되면 V1과 V2 Pod들이 추가되고 삭제되는게 순식간에 진행되므로, 10초라는 텀을 갖고 진행함을 의미
   - ReplicaSet은 Selector와 Pod의 Label로 매칭
     + Rolling Update가 진행 중인 순간의 경우, ReplicaSet의 Label은 해당 Pod로 연결될 수 있는가?
     + Label과 Selector의 역할을 보면, 그래야 되는데 연결되지 않는 이유는 Deployment가 ReplicaSet을 만들 때, 지정한 Selector만 가지고 ReplicaSet과 Pod의 계를 매칭하지 않고, 추가적인 Label과 Selector를 더 만들어서 매칭
