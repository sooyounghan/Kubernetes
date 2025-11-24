-----
### Pod - Container, Label, NodeSchedule
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/38c5fddf-4f7d-41ff-a1d2-e58735f68b02">
</div>

1. Pod
   - Pod 안에는 하나의 독립적인 서비스를 구동할 수 있는 컨테이너들이 존재
     + 이 컨테이너들은 서비스가 연결될 수 있도록 Port를 가지고 있으며, 한 컨테이너가 Port를 하나 이상 가질 수 있지만, 한 Pod 내 컨테이너들끼리 Port가 중복될 수 없음
     + 두 컨테이너는 한 Host로 묶여있다고 보면 되는데, 왼쪽 컨테이너에서 오른쪽 컨테이너로 접근할 때 localhost:8080으로 접근할 수 있음
   - 그리고 Pod가 생성될 때 고유의 IP 주소가 자동 할당되는데, Kubernetes Cluster 내에서만 이 IP를 통해서 접근할 수 있으며, 외부에서는 해당 IP로 접근 불가
     + 만약, Pod에 문제가 생기면 시스템이 이걸 감지하여 Pod를 삭제시키고, 다시 재생성을 하는데 이 떄, IP 주소가 변경 (휘발성이 있는 특징을 가짐)
   - 즉, 한 Pod에는 여러 컨테이너를 담을 수 있음

   - Pod Yaml 파일
     + name : pod-1
     + containers에는 container1 container2가 존재
       * container1는 이미지가 p8000이라는 이름이며, Port는 8000번으로 노출
       * container2는 이미지가 p8080이라는 이름이며, Port는 8080번으로 노출

3. Label
   - Pod 뿐만 아니라 모든 Object에 부착할 수 있는데, Pod에서 가장 많이 사용
   - 사용 이유 : 목적에 따라 Object들을 분류하고, 분류된 Object들만 선택하여 연결하기 위함
   - 구성 : Key와 Value가 한 쌍이며, 한 Pod에는 여러 개의 Label을 붙일 수 있음
   - 이렇게 사용 목적에 따라 Label을 잘 붙여 놓으면, 원하는 Pdo를 선택해서 사용 가능
   - Label Yaml 파일
     + Pod를 만들 때 , labels에 Key-Value 형식으로 내용을 넣을 수 있음
     + 추후 서비스를 만들 때, selector에 Key와 Value를 넣으면 해당 내용과 매칭되는 Label이 붙어 있는 Pod로 연결

4. Node Schedule
   - Pod는 결국 여러 노드들 중 한 노드에 적재
     + 적재되는 방법에는 직접 노드를 선택하는 방법과 Kubernetes가 자동으로 지정해주는 방법
     + 직접 노드를 선택하는 방법 : Pod에 Label을 부여한 것 처럼 Node에 Label을 부여하고, Pod를 만들 때, Node를 지정할 수 있음
       * Yaml 파일을 보면, Pod를 만들 때, nodeSelector라는 항목에 Node에 Label과 매칭하는 Key-Value를 넣으면 됨
     + Kubernetes의 Scheduler가 판단 해 지정하는 방법
       * 노드에는 전체 사용 가능한 자원량이 존재 : CPU와 메모리가 대표적
       * 예를 들어, Node1 노드에 몇몇 Pod가 들어가 남은 메모리 1GB이고, Node2에는 3.7GB의 메모리가 남았다고 가정
       * Pod를 생성할 때, 이 Pod에서 요구될 리소스의 사용량을 명시할 수 있는데, 이 Pod는 2GB의 메모리를 요구한다라고하면, Kubernetes가 Pod를 스케줄링함
       * 사용량을 넣는 이유 : 이를 설정하지 않으면 Pod 안에 있는 앱에서 부하가 생기면, 무한정 노드에 있는 자원을 사용하려 할 것이며, 그 노드에 있는 다른 Pod들은 자원이 없어 결국 죽게 됨
   - Pod Yaml 파일
     + resources는 requests가 메모리 2GB를 요구한다는 뜻
     + limits는 최대 허용 메모리가 3GB라는 뜻
       * 메모리의 경우 limits를 넘어버리면 바로 Pod 종료
       * CPU의 경우 직접 Pod를 종료시키지 않음
       * 이러한 이유는 각 자원에 대한 특성 때문으로, 프로세스들이 CPU 자원을 쓰는데 있어서 문제가 발생하지 않기 때문이지만, 메모리는 프로세스 간에 치명적 문제를 일으키므로 자원의 성격에 따라 Kubernetes가 다르게 판단
      
