-----
### Template, Reflication Controller, ReplicaSet, Selector
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/716d83f4-b8d9-4410-95d4-6174c474acc8">
</div>

1. Template
   - Controller와 Pod는 Service와 Pod처럼 Label과 Selector로 연결이 되므로, Label이 붙어있는 Pod도 존재하고, Selector로 매칭되는 Controller를 만들면 연결이 됨
   - Controller를 만들 때, Template으로 Pod의 내용을 넣게 되는데, Controller는 Pod가 죽으면 Pod를 재생성시킴
   - 따라서, Pod가 다운되면, 안의 Template으로 Pod를 새로 만들어주게 되므로, 이러한 특성을 이용해 App에 대한 업그레이드가 가능 (템플릿에 v2에 대한 Pod를 업데이트 후, 기존에 연결되어 있는 Pod를 다운시키면 컨트롤러는 템플릿을 가지고 Pod를 재생성하려고 하므로 새로 업그레이드 된 Pod가 만들어지면서 버전 업그레이드를 수동으로 할 수 있게 됨)
   - Yaml 파일
     + Pod에 labels이 설정되어 있음
     + Replication Controller를 만들 때, Selector를 통해 연결하면 연결 (selector.type: web)
     + template에 Pod의 내용이 들어가는데, 여기도 마찬가지로 Label을 지정해놔야 새로 만들어질 때 Controller와 연결

3. Reflicas
   - Replicas만큼 Pod의 개수가 관리되며, (현재 설정 1) Pod가 삭제되면 하나의 Pod만 재생성
     + 만약 이 수치를 3으로 늘리게 되면, 그 수만큼 Pod가 늘어나면서 Scale-Out
     + 마찬가지로, 이 파드들을 모두 삭제하면 Controller는 Replicas에 대한 개수만큼 3개의 Pod를 만들어줌
   - Yaml 내용
     + 개수를 늘려주게 되면 Scale-Out, 수치를 내리면 Scale-In
     + 위 두 기능은 Pod와 Controller을 따로 만들지 않고, 한 번에 만들 수 있으므로, 내용을 담아 Pod 없이 컨트롤러만 만들면, Controller는 replicas가 2라고 되어이는데, 현재 연결되어 있는 Pod가 없으므로 Template에 있는 Pod의 내용을 가지고 2개의 파드를 생성

5. Selector
   - Replication Controller Selector는 Key와 Label이 같은 Pod들을 연결하며, 둘 중 하나라도 다르면 연결하지 않음
   - ReplicaSet Selector는 두 가지 추가적인 속성이 존재
     + MatchLabels : Replication Controller와 같이 Key-Value가 모두 같아야 연결
     + MatchExpressions : Key-Value를 더 디테일하게 Control
       * 예를 들어, Key에 ver, Operator에 Exist : Value는 다르지만 Label의 키가 ver인 모든 Pod 선택
   - Yaml 파일
     + selector에 바로 key, value가 들어가지 않고, matchLabels, matchExpressions가 존재
     + matchLabels에는 Key-Value가 들어감
     + matchExpressions에는 상세한 설정이 들어감
     + operator
       * Exists : 키를 정하고, 그에 맞는 키의 값을 가지고 있는 Pod들을 연결
       * DoesNotExists : 키에 똑같이 A라고 설정을 하면 키에 A가 들어가지 않은 다른 Pod만 선택
       * In : Key와 Value 수 지정 가능 (키를 A라고 설정하고, Value 수를 2, 3으로 설정하면 키가 A인 Pod 중 Value가 2, 3인 두 Pod만 선택)
       * NotIn: In과 반대 기능 (2와 3이 들어가 있지 않은 Pod 선택)

6. ReplicaSet은 다양한 방식으로 Pod를 선택할 수 있는 Selector 옵션 제공
