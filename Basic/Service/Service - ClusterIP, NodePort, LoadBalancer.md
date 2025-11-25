-----
### Service - ClusterIP, NodePort, LoadBalancer
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/fb21c08a-2d7b-469b-b4a7-b34b59efedbe">
</div>

1. ClusterIP : Pod의 접근의 가장 일반적인 방식
   - Service는 기본적으로 자신의 Cluster IP를 가지고 있음
   - 이 Service를 Pod에 연결시켜 놓으면, 서비스의 IP를 통해서도 Pod에 접근 가능
     + Pod는 Kubernetes에서 시스템 장애 또는 성능 장애 등 언제든지 장애가 발생할 수 있고, 재생성하기 위해 설계된 Object
     + 하지만, 이럴 경우 재생성된 Pods의 IP는 변경되므로, Pod의 IP는 신뢰성이 떨어짐
     + 하지만, Service는 사용자가 직접 지우지 않는 한 삭제되거나 재생성되지 않음
     + 따라서, Servic의 IP로 접근하면, 항상 연결되어 있는 Pod에 접근 가능 (Service 사용 목적)
   - Pod와 동일하게 Kubernetes Cluster 내에서만 접근 가능한 IP : 클러스터 내에 다른 모든 Object들이 접근을 할 수 있지만, 외부에서는 접근 불가
   - 여러 개의 Pod를 연결시킬 수 있는데 여러 개의 Pod를 연결시켰을 때, Service가 트래픽을 분산해서 Pod에 전달
     + type: ClusterIP (Optional) - 생략하면, 기본값이 ClusterIP이므로, 이를 사용할 때는 넣지 않아도 됨
     + 9000번 포트로 들어오는 포트에 대해 8080 포트로 연결됨
   - 즉, 외부에서 접근할 수 없고 클러스터 내에서만 사용하는 IP 
     + 따라서 이 IP에 접근할 수 있는 대상은 클러스터 내부에 접근할 수 있는 인가된 사용자 (운영자)만 가능
     + 내부 Dashboard에서 사용
     + Pod의 서비스 상태 디버깅 목적

2. NodePort
   - Service는 기본적으로 ClusterIP가 할당이 되어 ClusterIP와 같은 기능이 포함
   - Kubernetes Cluster에 연결되어 있는 모든 노드한테 똑같은 포트가 할당이 되어서 웹으로부터 어느 노드라도 그 IP로 접속하면 해당 Service로 연결
   - Service는 기본 역할인 자신에게 연결되어 있는 Cluster에 트래픽을 전달
   - 주의사항 : Pod가 있는 노드에만 Port가 할당되는 것이 아닌 모든 노드에 Port가 만들어짐
   - type: nodePort로 지정 가능하며, nodePort는 30000 ~ 32767번대 사이에서만 할당 가능 (Optional Value이므로 넣지 않으면 자동 할당)
   - Node 1의 IP로 접근하더라도, 이 Service는 다른 Node에 있는 Pod에게 트래픽 전달 가능
     + 즉, Service 입장에서는 어떤 Node한테 온 트래픽인지 상관없이 그냥 자신에게 부속된 Pod들에게 트래픽을 전달 가능
     + 하지만 만약, yaml에 externalTrafficPolicy: Local로 설정하면, 특정 노드 포트의 IP로 접근을 하는 트래픽은 해당 노드 위에 있는 Pod한테만 트래픽 전달
   - 물리적인 호스트의 IP를 통해 Pod에 접근
     + 호스트 IP는 보안적으로 내부망에서만 접근을 할 수 있게 네트워크를 구성하므로, 클러스터 밖에 있지만, 내부망 안에서 접근해야 될 때 사용
     + 데모, 일시적인 외부 연동용 또는 임시 연결 등에 사용 

3. Load Balancer
   - 각 노드에 트래픽을 분산시켜주는 역할
   - 한 가지 문제점 : Load Balancer에 접근을 하기 위한 외부 접속 IP 주소는 Kubernetes를 생성할 때 기본적으로 생기지 않음
     + 즉, 별도로 외부 접속 IP를 할당해주는 플러그인이 설치가 되어 있어야 IP 생성 : 해당 IP로 외부에서 접근이 가능
   - type: LoadBalancer로 지정
   - 실제적으로 외부에 서비스를 노출 시키는 용도로 사용 : 내부 IP를 노출하지 않고, 외부 IP를 통해 안정적 서비스 제공 가능
