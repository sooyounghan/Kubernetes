-----
### Volume - Dynamic Provisioning, StorageClass, Status, ReclaimPolicy
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/c85a5f9a-f04a-4e2b-974a-6af7b44874c6">
</div>

1. Volume : 데이터를 안정적으로 유지하기 위해 사용
   - 그러기 위해서는 실제 데이터는 Kubernetes Cluster와 분리가 되서 관리
   - 내부망과 외부망에서 관리하는 경우
     + 외부망 : Amazon WebService나 Google CloudService, Microsoft Azure와 같은 Cloud Storage와 Kubernetes Cluster와 연결해서 사용 가능
     + 내부망 : Kubernetes를 구성하는 노드들이 있는데, 기본적으로 Kubernetes에서 이 노드들의 실제 물리적인 공간의 데이터를 만들 수 있는 Host Path나 Local Volume이 존재하고, 별도의 On-Premise Storage Solution들을 Node에 설치 가능
       + Stroage OS나 Ceph, ClusterFS와 같은 종류가 있고, 해당 솔루션들이 알아서 Node 자원을 이용해 Volume을 관리
       + NFS를 사용해 다른 서버를 볼륨 자원으로 사용 가능
       + 이 밖에도 많은 종류의 볼륨 서비스들이 존재

   - Kubernetes Cluster 밖에 실제 볼륨들이 마련되어 있다면, 관리자는 PV를 만드는데 저장 용량과 액세스 모드를 정하고 볼륨을 선택해서 연결
     + 사용자는 원하는 원하는 용량과 액세스 모드로 PVC를 만들면 Kubernetes가 적절한 PV와 연결해주고 PVC를 Pod에서 사용

   - Access Mode : 한 노드에서 읽기, 쓰기가 되거나 여러 노드에 읽기만 되고 또는 읽기, 쓰기가 되는 세 종류가 존재
     + Kubernetes는 각 볼륨마다 실제로 지원되는 것이 다름

<div align="center">
<img src="https://github.com/user-attachments/assets/464fabbb-2065-423d-be66-507d45a15ec0">
</div>

2. Dynamic Provisioning : 사용자가 PVC를 만들면 PV를 생성해주고 실제 볼륨과 연결해주는 기능
   - 모든 PV에는 각각의 상태가 존재하는데, 이 상태를 통해 PV가 PVC에 연결되어 있는 상태인지 아니면 끊겼는지, 에러가 발생했는지 등 알 수 있고, PV를 삭제하는 부분에 있어 정책적인 요소가 필요할 수 있음
   - 사전작업으로 이를 작업해주는 Storage Solution을 선택해서 설치해야 함
   - 설치가 되면 Service나 Pod 등 여러 Object 등이 생성이 되지만, StorageClass라는 Object가 가장 중요함
     + 이를 사용해 동적으로 PV를 만들 수 있는데, PVC를 만들 떄 StorageClass name이 존재하는데, 만들어져 있는 StorageClass name을 넣으면 자동으로 StorageOS Volume을 가진 PV가 생성
     + 또한, Storage Class를 추가로 만들 수 있고, Default 설정이 가능한데, 생성해놓으면 PVC에 Storage Class 이름을 생략했을 때, Default Storage Class가 적용되어 PV 생성

3. Status, ReclaimPolicy
   - Status : 최초 PV가 생성되었을 때 Available 상태이며, PVC와 연결이 되면 Bound 상태
     + PV를 직접 만드는 경우에는 아직 Volume의 실제 데이터가 만들어진 상태는 아니고 Pod가 PVC를 사용해 구동이 될 때 실제 Volume이 생성
     + Pod의 서비스가 유지가 되다가 Pod가 삭제될 경우 PVC와 PV는 아무런 변화가 없어도 Pod가 삭제되더라도 데이터는 문제가 없음
     + PVC를 삭제해야만 PV와 연결이 끊어지면서 PV의 상태는 Released 상태
     + 이러한 과정 중에 PV와 실제 데이터 간 연결 문제가 생기면 Failed 상태로 변함

   - ReclaimPolicy : PVC가 삭제됐을 때 상황에 대해 PV가 설정해 놓은 것
     + Retain, Delete, Recycle 3가지 상태 존재
     + Retain : PVC가 삭제되면 PV의 상태는 Release가 되는데, PV를 만들 때, 별도로 설정하지 않을 경우 기본 정책으로, 실제 볼륨 데이터는 유지가 되지만, 해당 PV를 다른 PVC로 연결 불가 (따라서, 재사용이 불가하므로 PV를 수동으로 만든 것처럼 삭제도 수동으로 실시해야 함)
     + Delete : PVC를 삭제하면 PV와 같이 삭제되며, Storage Class를 상요해서 자동으로 만들어진 PV의 경우 기본 정책 (Volume의 종류에 따라 실제 데이터가 삭제되거나 되지 않을 수 있음)
     + Recylce : PV의 상태가 Available이 되면서 PVC에서 다시 연결할 수 있는 상태 (현재는 Deprecated)
       * 실제 데이터가 삭제가 되면서 PV를 재사용할 수 있음
