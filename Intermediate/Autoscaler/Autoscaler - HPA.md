-----
### Autoscaler - HPA
-----
<img width="1524" height="749" alt="image" src="https://github.com/user-attachments/assets/1f2d354d-76e3-4608-8e36-f5674a473c69" />

1. Kubernetes Autoscaler : HPA(Pod의 개수를 늘림), VPA(Pod의 리소스를 증가), CE(클러스터에 노드를 추가)
2. HPA
   - 컨트롤러가 있으며, Replicas의 수치에 따라 Pod가 만들어져 운영되고 있음
   - 서비스도 연결이 되서 모든 트래픽이 이 Pod로 흐르고 있음
   - 트래픽이 많아져서 어느 순간 Pod 내 모든 리소스를 모두 사용하게 되었고, 조금 더 트래픽이 증가하면 Pod는 죽을 수 있음
     + 만약 사전에 HPA를 만들고, 이 컨트롤러에 연결했다면, HPA가 Pod의 리소스 상태를 감지하고 위험한 상황이 오면 컨트롤러의 Replicas를 증가시킴
     + 따라서, 컨트롤러는 Pod를 하나 더 생성하고 Pod는 수평적으로 증가하는 것을 Scale-Out
   - 반대로 트래픽이 감소해서 리소스 사용량이 줄게 되면 Pod는 삭제되는데, Scale-In
   - Pod가 증가했기 때문에 각 트래픽이 50%씩 분산이 되어서 자원 사용량은 똑같이 배분이 되고, 안정적 서비스 유지 가능
   - 권장되는 조건
     + HPA는 장애가 발생할 수 있는 상황을 고려해 만들어진 기능 : 장애 상황에서는 빠른 복구가 중요하므로 가동이 빠르게 되는 App에서 사용하는 것이 권장
     + Stateless App : Stateful App에서 사용하게 되면, App은 Pod마다 각 역할이 있는데, 마스터 역할을 하는 Pod와 Slave 역할을 하는 Pod 중 HPA가 어떤 Pod를 늘려야 하는지 판단할 수 없음
        * 그러므로, 양적인 증가 및 감소를 해도 문제가 없는 Stateless App 권장

3. VPA
   - 컨트롤러에 Replicas 1로 Pod가 하나 만들어져 있고, Pod는 메모리가 1GB, CPU가 1코어로 정의
   - 이 리소스를 모두 사용되는 상황 발생
     + 그냥 두면 Pod가 적지만, VPA을 컨트롤러에 설정했으면 VPA가 리소스 상태를 감지하고 있다가 이를 인지하고 Pod를 Restart시키면서 리소스를 증가시킴
     + 리소스의 양을 수직으로 증가 : Scale-Up
     + 리소스의 양을 수직으로 감소 : Scale-Down
   - Stateful App에 대한 Auto-Scaling을 할 때 사용
   - 주의사항 : 하나의 컨트롤러에 VPA와 HPA을 동시에 설정하면 기능이 작동하지 않음

4. CA
   - 클러스터에 있는 모든 노드에 자원이 없을 경우 동적으로 worker-node를 추가시켜줌
   - 방식
     + 노드 2개가 있고 Pod들이 운영 중인데, Pod를 만들면 Scheduler가 할당해주는 노드에 배치
     + worker-node들의 자원이 모두 소모가 된 상태에서 Pod를 하나 더 만들려고 하면, 스케줄러는 더 이상 어느 노드에도 배치할 수 없다는 것을 확인하고 CA에게 worker-node를 생성해달라고 요청
     + CA를 사전에 특정 Cloud Provider에 연결했다면 요청이 들어와을 때, 해당 Provider에 Node를 하나 더 만들어주고, 스케줄러는 Pod를 여기에 배치
     + 운영이 되다가 기존 사용하던 Pod들이 없어져서 로컬 노드에 자원이 남게 되면, Cloud Provider는 사용 시간에 따라 돈을 받으므로, 필요할 때만 사용하는 것이 좋은데, 스케줄러는 로컬 노드에 자원이 여유있다는 것을 감지하고 CA한테 노드를 삭제해달라고 요청
     + 노드가 삭제가 되면서 Pod는 로컬 노드로 옮겨짐

5. HPA Architecture
<div align="center">
<img src="https://github.com/user-attachments/assets/edd4a0aa-13f9-4b52-b3cf-c3390343ec51">
</div>

   - 마스터 노드와 워커 노드들이 있고, 마스터의 Contorl Plane Component라고 하여 Kubernetes의 주요 기능을 하는 컴포넌트들이 Pod의 형태로 띄어져서 작동되고 있는데, 이 중 Controller Manager는 사용하고 있는 컨트롤러(Deployment, ReplicaSet, DaemonSet, HPA, VPA 등)의 기능들이 스레드 형태로 작동
   - kube-apiserver가 존재하는데, Kubernetes 노드 통신의 길목 역할
     + 사용자가 쿠버네티스에 접근했을 때 이용
     + Kubernetes 내 컴포넌트들 조차 DB에 접근 또는 타 컴포넌트들을 호출할 때 이 API를 통해서 통신

   - Worker Node Componet : Kuberentes를 설치할 때 kubelet가 설치 (각 노드마다 설치가 되고, 노드를 대표하는 에이전트 역할을 하는데, 자신의 노드에 있는 Pod를 관리하는 역할)
     + kubelet이 직접 컨테이너를 만드는 것이 아닌, Controller Runtime이 실제 컨테이너를 생성하고 삭제하는 역할을 하는 구현체ㅈ가 존재
     + 현재 Docker를 설치해서 사용하므로 이 외에도 컨테이너를 만들어주는 RKT, Core OS 등 존재

   - 예) 사용자가 ReplicaSet을 만들 때 과정
     + ReplicaSet을 담당하는 스레드는 Replicas가 1이라고 할 때, Pod를 하나 만들어달라고 kube-apiserver를 통해 kubelet에게 요청
     + kubelet은 Pod는 Kuberentes의 개념이며, 이 안에 컨테이너만 빼서 만들어달라고 Docker에게 요청하고, Docker가 노드 위에 컨테이너를 만들어줌
     + 이 상태에서 HPA가 Pod의 성능 정보를 알게 되는 방법
       * Resource Estimator인 cAdvisor가 Docker로부터 메모리와 CPU에 대한 성능 정보를 측정하는데, 이 정보를 kubelet을 통해 가져갈 수 있게 해놓음
       * AddOn Component로 Metrics 서버를 설치해야 하는데, 이를 설치하면 Metrics 서버가 각 노드들에 있는 cubelet한테 메모리와 CPU 정보를 가져와서 저장하며, 데이터들을 다른 컴포넌트에서 사용할 수 있도록 kube-apiserver에 등록
       * HPA가 CPU와 메모리 정보를 kube-apiserver를 통해 가져갈 수 있게 되고, HPA는 15초마다 확인을 하고 있다가 리소스 사용률이 높아졌을 때, ReplicaSet를 증가
       * kubectl top 명령을 치면, Resource API를 통해 Pod나 Node의 현재 리소스 상태를 조회해볼 수 있음
       * 추가적으로 프로메테우스를 설치하면 단순 메모리나 CPU 외에도 다양한 Metric 정보를 수집할 수 있음 (예를 들어, Pod로 들어오는 패킷 수 또는 Ingress로 들어오는 요청 양들에 대한 Metric 정보 제공)
       * HPA는 이 정보를 트리거로도 컨트롤러의 Replicas를 조절 가능

6. HPA
<div align="center">
<img src="https://github.com/user-attachments/assets/fabf82cc-2108-4bf7-b34b-25af468c11ce">
</div>

   - Replicas를 2로 Deployment를 만들면, ReplicaSet이 만들어지면서 Pod가 2개 생성
   - Pod의 resources.limit과 request가 그림과 같이 설정되어 있는 상태에서 Scale-Out, Scale-In을 해주는 HPA 생성
     + 설정으로는 먼저 Target Controller을 지정하는 부분이 있고(target), 증가되는 ReplicaSet에 대해 minReplicas와 maxReplicas를 지정해놔야 함
     + metrics은 Metrics 정보의 어떤 조건을 통해서 Replicas를 증가시킬 것인지에 대해 Type이라고 해서 Resource라고 선택하면, 이 Pod의 resources 부분을 가리킴
       * 세부적으로는 name에 CPU 또는 메모리를 측정할 것인지 설정 가능
       * 마지막으로 어떤 조건으로 증가시킬지에 대한 부분으로, 가장 기본으로 사용되는 옵션은 Utilization이 있고, averageUtilization을 50%라고 했다면 이 Pod의 Request 값을 기준으로 현재 사용자원의 평균 50%가 넘으면 ReplicaSet을 증가시키고, 이 수치가 넘으면 공식을 통해 한 번에 몇 개 증가시킬지 결정
       * 리소스를 CPU라고 가정하고, 이 두 Pod의 평균이 200이고, Utilzation은 50%이므로, 실제 CPU 사용률이 100이 넘게 되면 HPA는 Replicas를 증가시키게 됨
       * Scale- Out 되는 예 : 현재 평균 CPU가 300이 되었다고 가정하고, limits.cpu는 500이므로 Pod는 죽지 않음
         * 몇 개의 Pod를 증가시킬지 공식을 대입하면, 현재 Replica는 2에 현재 평균 CPU를 곱하고 지정한 target 값으로 나누면 HPA가 증가시킬 Replicas 값
       * Scale-In 되는 예 : 6개 Pod로 Scale-Out 된 상태에서 평균 CPU가 50으로 떨어졌다고 가정하고 공식을 적용하면 변경시킬 Replicas의 값은 3
         * CPU가 50이면 계산에 의해서 나온 Replicas의 수 만큼 Pod를 감소시키고, 이 값은 Minimum이므로, 더 이상 Pod는 줄지 않게 됨
       * 그리고 이 type의 종류에는 %가 아닌 실제 평균 수치 값을 넣는 Average Value와 Value가 존재

   - Metric Type 종류
     + Pod에 Ingress가 설정되어있다고 가정할 때, Custom API를 통해 Type이 Pods이면 Pod에 들어오는 패킷 수 또는 파드와 관련된 정보로 Scale-In / Out 가능
     + Ingress와 같은 다른 Object에 대한 Metric 정보는 type을 쓰면 됨
     + Cusotm API를 쓰려면, 프로메테우스와 같은 플러그인이 설치되어야 함
