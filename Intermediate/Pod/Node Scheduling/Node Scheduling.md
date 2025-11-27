-----
### Pod - Node Scheduling
-----
1. Pod는 기본적으로 Scheduler에 의해 노드가 할당이 되지만 사용자의 의도에 의해서 노드를 지정할 수 있고, 운영자가 특정 노드를 사용하지 못하도록 관리할 수 있음
<div align="center">
<img src="https://github.com/user-attachments/assets/fe8fe2f0-c630-4939-8ac9-52b4059ac9de">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/cdbc2a2d-ce0a-4836-b718-9c6bdf621685">
</div>

2. Node 선택 사용 상황
   - NodeName, NodeSelector, NodeAffinity는 Pod를 특정 노드에 할당되도록 선택하기 위한 용도로 사용
     + 노드가 위 그림처럼 존재하고 노드들마다 가용한 CPU 자원들이 있다고 가정
     + Label 또한 Node1부터 3까지 서버가 한국에 있어 Labeling을 통해 Grouping을 하고, 노드 4와 노드 5는 미국에 있어 Labeling
     + 이 상태에서 Pod를 하나 만들면 Kubernetes Schedular는 CPU 자원이 가장 많은 노드에 Pod를 할당
   - NodeName : 하지만 내가 원하는 노드를 선택하고 싶을 때, NodeName을 사용하면 Schedular와 상관없이 바로 해당 노드의 이름으로 Pod 할당
     + 명시적으로 Node를 할당할 수 있다는 점이 좋아보이지만, 실제 상용 환경에서는 Node가 추가되고 삭제되면서 이름이 계속 변경될 수 있으므로 잘 사용하지 앟음
   - NodeSelctor : 특정 노드를 지정할 때 권장되는 방법
     + Pod에 Key와 Value 값을 달면, 해당 Label에 설정된 노드에 할당
     + 하지만 Label 특성 상 여러 노드의 같은 Key와 Value을 가진 Label을 설정할 수 있으므로, NodeSelector를 위 그림처럼 주게 되면, Schedular에 의해 두 노드 중 자원이 많은 노드로 Pod 할당
     + 하지만, Key와 Value가 정확히 매칭이 되는 노드만 할당되며, 매칭이 되는 Label이 없다면 Pod는 어느 노드에도 할당되지 않아 에러 발생
   - NodeAffinity : NodeSelector의 문제점을 보완해서 사용하는 방법
     + Pod에 Key만 설정해도 해당 그룹 중 Schedular를 통해 자원이 많은 노드에 Pod가 할당이 됨
     + 해당 조건에 맞지 않는 Key를 가지고 있더라도 Schedular가 판단해서 자원이 많은 노드에 할당이 되도록 옵션을 줄 수 있음

3. Pod 간 집중 / 분산 - Pod Affinity, Pod Anti-Affinity
   - 여러 Pod들을 한 Node에 집중해서 할당 또는 Pod들 간 겹치는 노드 없이 분산을 해서 할당하는 방식
     + Pod Affinity 예) 웹과 서버가 있는데, 두 Pod는 한 PV에 연결되어 있고, 이 PV가 HostPath를 쓴다고 할 떄, 이 두 Pod는 같은 노드 상에 있어야만 문제가 없게 됨
       * 따라서, 두 Pod를 같은 노드에 할당하려면, Pod Affinity를 사용해야 하는데, 처음 Web Pod가 Schedular에 의해 특정 한 서버에 할당이 되면, 그 HostPath에 PV가 생성
       * 따라서, 서버 Pod가 웹 Pod에 있는 노드에 들어가게 하려면, Pod 생성 시 PodAffinity 속성을 넣고 웹에 있는 Label 지정
       * 서버 Pod는 웹 Pod에 있는 노드로 할당

   - Pod Anti-Affinity : 두 Pod는 Master가 죽으면 Slave가 백업을 해줘야하는 관계일 떄, 이 Pod가 같은 노드로 들어갈 경우 그 노드가 Down이 되면, 둘 다 장애가 되므로 서로 다른 노드의 Scheduling이 되어야 함
     + 따라서, Master Scheduler에 의해서 어느 한 Node에 들어가고 Slave Pod를 만들 때, Anit-Affinity를 부여해 Master Pod의 Key-Value 설정을 하면, 이 Pod는 Master와 다른 노드에 할당

4. Node에 할당 제한 - Toleration / Taint
   - 특정 노드에는 아무 Pod나 할당이 되지 않도록 제한하기 위해 사용
   - Node 5는 높은 사양 그래픽 카드를 요구하는 앱을 돌리는 용도로 GPU를 설정해놨을 때, 운영자는 taint를 설정
   - 일반적인 Pod들은 Scheduler가 이 노드에 할당시키지 않고, 직접 Pod가 Node 5를 지정을 하고 할당하려 해도 되지 않음
   - 이 노드에 할당이 되려면 Pod는 toleration을 설정해야 할당 가능

<div align="center">
<img src="https://github.com/user-attachments/assets/44aa8573-568f-4c6f-935a-97145109e376">
</div>

5. NodeAffinity
   - matchExpressions : 여러 조합으로 Pod 선택 가능
     + 이 속성을 통해 Pod는 Node들을 선택 가능
     + Pod를 Key가 ver1인 그룹 안에 할당하고 싶을 때, key: kr과 같이 사용
     + 스케줄러가 이 두 노드들 중 자원이 많은 쪽으로 Pod를 배치
     + Operator 종류
       * Exists, DoesNotExists, In, NotIn은 Replicas와 동일
       * GT, NT는 지정한 Value보다 값이 크거나 작은 노드를 선택하는 옵션
     + required, prefrred
       * 노드 2개가 있을 때, 만약 NodeAffinity로 required 속성을 가진 Pod가 노드에는 없는 키를 가지고 있다면, Pod는 Node에 절대 스케줄링 되지 않음
       * 만약, 키가 ch인 Node가 있으면 선호를 할 뿐이지, 그 노드에 할당되어야 하는 건 아닐 경우, preferred 옵션을 설정하면, 해당 키가 없더라도 적절한 노드에 할당
       * preferred는 weight라는 필수값이 존재 : Key가 다른 Level을 가진 두 노드가 있고, CPU는 50:30으로 Node1와 Node2에 할당
       * preferred 속성을 가진 Pod를 만드는 데, 두 노드에 Key가 모두 있으므로, 두 노드 중 CPU 자원이 많은 Nod1에 할당이 되는데, wait 옵션을 설정하면, 선호도에 대한 가중치를 주는 것으로, 이 Pod는 Key가 us 또는 kr인 노드에 할당될 수 있지만, Key가 kr인 노드에 가중치를 좀 더 주겠다고 설정하는 것
       * 따라서, 스케줄러는 최초 CPU 자원을 보고 점수를 매겨 Node 1에 할당시키려고 하지만, Pod의 가중치가 합산되어 다시 점수를 매기고, 매긴 점수가 높은 쪽으로 할당



6. PodAffinitiy
   - 노드들이 있고, 두 노드는 a-team이라는 Key를 가진 Label을, 그리고 남은 두 노드에는 b-team이라는 Key를 가진 Label 설정
   - Pod-Affinity
     + Web이라는 Label을 가진 Web Pod가 스케줄러에 의해 노드 1에 할당
     + Web Pod가 같은 노드에 Pod를 넣으려면, Pod에 podAffinity라는 속성으로 matchExpression 설정 : Pod의 Level과 매칭되는 조건을 찾음
     + Web Pod와 같이 Node 1에 할당이 될 수 있는데, topologyKey는 노드의 Key를 봄 : 즉, matchExpressions에 있는 조건의 라벨을 가진 Pod를 찾지만, 노드의 키가 A-team에 있는 범위 내에서만 찾겠다는 의미
     + 만약, Web Pod가 Node 3에 들어가있으면, 이 Server Pod는 Pending 상태가 되고, 해당 조건을 만족할 때까지 노드에 할당되지 않음
     + required, preferred 옵션이 적용
   - Pod-AntiAffinity
     + Master Label을 가진 Master Pod가 Node 4에 Scheduling이 되었을 때, 다른 노드에 할당하기 위해 Slave Pod는 antiAffinity를 설정하고, matchExpressions로 Master Pod에 Label을 달면, Pod가 있는 Node에 할당되지 않음
     + 마찬가지로 Topology Key를 주어 b-team이라는 Key를 가진 노드들 범위로 한정 가능

<div align="center">
<img src="https://github.com/user-attachments/assets/a60722a4-841c-41a9-9008-1a6b08668f22">
</div>
7. Taint, Toleration
   - Node 1은 타 노드들과 달리 GPU가 구성되어 있음
   - 일반적인 Pod들이 이 Node에 스케줄링 되는 걸 막으려면 tainit를 Node에 설정
     + taint에는 taint를 식별하는 Label인 Key, Value
     + effect라는 옵션이 있어, NoSchedule을 주면, 타 Pod들이 이 Node에 할당되지 않음
       * PrerferNoSchedule은 가급적 스케줄링이 안 되도록 설정하는 것이며, Pod가 다른 노드에 배치할 수 없는 상황이라면, taint가 붙은 노드에는 할당을 허용해줌
       * 따라서, Node 1에는 일반적인 노드들이 절대 할당되지 않음
   - 하지만, 실제 GPU를 사용해야되는 Pod라서 Node 1에 할당이 되어야 한다면, Pod를 만들 떄 Toleration을 부여
     + Key, Operation, Value가 있으며, 이 조건이 taint의 Label과 맞아야 함
     + Operator는 Equal과 Exists만 존재
     + effect 종류까지 모두 매칭이 되어야만, Node 1에 할당이 되며, 하나라도 틀리면 Toleration을 설정해도 불가
     + 💡 Toleration 내용으로 매칭이 되는 Taint가 있는 노드를 찾는다고 생각할 수 있는데, Pod가 Nod1에 스케줄링 됐을 때, Pod에는 Node 1의 Taint와 매칭되는 Toleration이 있으므로 노드에 들어갈 수 있는 조건이 부합되는 것
     + 💡 즉, Pod 상태에서는 다른 노드에도 스케줄링이 될 수 있으므로, Pod의 별도의 NodeSelector를 설정해 Pod가 Node 1에 배치될 수 있도록 해야 됨
   - Taint의 NoExecute, NoSchedule과 차이점
     + Pod 1이 Node 2에 할당이 되서 운영이 되고 있는데, NoSchedule 효과를 가진 Taint를 Node에 설정할 경우
     + NoSchedule 옵션은 모두 Pod가 최초 노드를 선택할 때만 고려되는 조건이므로, 이미 Pod가 Node에 연결된 상태에서 Node Label을 바꾸거나 noExecute 옵션을 taint로 설정한다고 해도 Pod가 삭제되거나 하지 않음 (즉, 마스터 노드에 기본적으로 설정되어 있어, Pod를 만들 때 Master에 할당이 되지 않도록 함)
     + 하지만, Taint의 NoExecute 옵션은 Taint를 설정하게 되면, Pod 2가 삭제 (만약, ReplicaSet에 의해 Pod가 운영 중인데 Node에 장애가 발생하면, Kubernetes는 해당 노드에 있는 Pod들이 정상적으로 동작하지 않을 수 있으므로, noExecute의 Taint를 자체적으로 Heading 노드에 설정하여 ReplicaSet은 자신의 Pod가 하나 없어져도, 다른 노드에 Pod를 다시 만들어서 서비스가 잘 유지되도록 해줌)
     + Node 3에 Taint가 설정되어 있어도, Pod가 삭제되지 않게 하려면 Taint의 내용과 매칭되는 Toleration을 설정하면 됨
   - toleartionSeconds : 없으면 삭제가 되지 않지면, 설정하면 해당 시간 후 삭제 (60으로 설정했으므로 Pod 3은 Node 3에 Taint가 들어오면 60초 후에 삭제)
   
