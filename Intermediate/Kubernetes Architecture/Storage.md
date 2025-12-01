-----
### Storage - File, Object, Block Storage
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/7d1da337-1aed-45e2-8302-7220d0777ea5">
</div>

1. PV 생성 방법
   - PV를 만들고 PVC를 바로 연결하는 방법
   - Storage Class를 Grouping 목적으로 사용하는 방법으로, Storage Class를 그룹의 개념으로 만들어놓고, PV를 만들 때 소속 그룹을 설정하며, PVC를 생성할 때는 Storage Class를 지정하면 PVC는 해당 그룹에 속해있는 PV들 중 자신의 스펙에 맞는 PV와 연결
   - Storage Class를 통해 동적으로 PV를 생성하는 방법 : 사전에 Storage Class를 생성할 때 Provisioner를 지정해야 하고, PVC를 생성할 때 해당 Storage Class를 지정하면 Provisioner가 PV를 만들어주면서 PVC는 PV에 연결
   - capacity : 용량을 설정하는 부분
   - accessModes : Access 모드를 지정하는 부분
   - Volume Plugin : PV의 실질을 결정하는 옵션
     + hostPath : PV를 Worker Node에 지정된 Path를 PV에 대한 볼륨으로 사용하겠다는 의미
     + NFS : 웹 스토리지가 NFS 서버로 구성되어 있다면, PV의 Volume Plugin을 NFS로 지정하게 되면, 이 PV는 외부에 있는 NFS 서버와 연결할 수 있는 NFS 클라이언트 역할을 하게 됨 (Kubernetes가 OS에 설치되어 있는 NFS 클라이언트 기능을 이용하므로 가능, 만약 OS에 NFS 클라이언트가 설치되지 않았다면 이 기능은 동작하지 않음)
     + 클라우드 서비스 볼륨에 연결하고 싶다면, 해당 서비스에 맞는 볼륨 플러그인들이 있고, 클라우드 서비스들 뿐 아니라 자신의 볼륨 솔루션이 설치가 되어있다면 PV를 통해 해당 솔루션과 바로 연결 가능 : 각 서비스 플랫폼이나 유명 솔루션 업체들에서 자신들의 볼륨과 연결할 수 있는 플러그인들을 Kubernetes Open Source에 기여했으므로 바로 연결 가능

   - CSI (Container Storage Inteface) : 자신의 볼륨 솔루션에 대한 플러그인들을 Kubernetes에 반영 또는 미반영되었더라도 볼륨의 최신 버전이 나왔을 때, 그 최신 버전을 지원할 수 있는 볼륨 플러그인을 바로 적용하고 싶을 때 사용

<div align="center">
<img src="https://github.com/user-attachments/assets/6eb76b9d-99a7-4117-ac8a-366619933bc6">
</div>

2. Access Mode
   - 각 볼륨들마다 Storage Type이 존재
     + NFS는 전형적인 File Storage
     + Cloud Service는 Volume Solution들이 있으므로, 대부분 Block Storage, File Storage, Object Stroage 모두 지원
     + 또한, Kuberenetes에 Plugin되어 있는 Third-Party Vendor들은 대부분 File Storage 지원
     + CSI 같은 Ceph Solution 이라고 하더라도, 컨테이너 베이스로 Volume Solution을 설치할 경우 모든 스토리지 타입이 제공 가능하며, 다른 솔루션들은 각자의 특성에 맞는 스토리지 타입 제공하므로, 각 볼륨 솔루션마다 솔루션이 어떤 타입의 스토리지인지 기본적 인지를 하고 있어야 하고, 스토리지에서 지원을 하는 Access Mode는 해당 가이드에서 확인을 해서 실제 제공하는 기능을 사용해야 함

   - File Storage와 Object Storage는 RWO를 지원하므로 한 노드 위에 여러 Pod에서 연결이 될 수 있고, RWM과 RWO도 지원할 수 있으므로 다른 노드에 있는 Pod들에서도 연결이 됨
   - Block Storage는 기술적으로 특정 한 노드에 스토리지를 Mounting하므로 한 노드 위에 Pod가 몇 개 있어도 연결이 되는 건 문제가 없지만, 다른 노드 위에 있는 Pod와 함께 연결이 될 수 없음
     + 즉, Node 3에 있는 Pod가 Node 2로 갔을 경우, 기존 노드와 연결을 끊고, 새 노드와 연결이 되는 건 가능하지만 두 노드와 이 블록 스토리지가 동시에 연결되는 건 불가능
     + 따라서, PV의 액세스 모드 지정 시에는, RWO (ReadWriteOnce) 옵션만 지원
   - 일반적으로 File Storage와 Object Storage는 Pod가 어떤 노드에 있건 연결이 가능해서 공유 데이터으로 사용
     + 그 중, Object Storage는 백업용이나 이미지 저장용으로 주로 사용하여, Deployment에 PVC를 설정할 때 자주 사용
     + Block Storage는 DB 데이터용으로 주로 사용 : Node와 Block Storage가 Mounting 되어 있으므로, 빠른 Read-Write가 가능
       * 또한, StatefulSet은 Pod가 만들 때마다 PVC를 새로 생성하기 때문에 적합하므로, DB 솔루션들을 보면 Block Storage를 사용해 StatefulSet으로 배포를 하는 경우가 많음

<div align="center">
<img src="https://github.com/user-attachments/assets/833bd24c-b777-4410-81ee-51698dd9d807">
</div>

3. Volume Solution과 CSI Plugin을 모두 설치했을 경우, Node들을 위해 Pod들이 생성
   - Control Plugin Pod : deployment로 주로 만들어지고, CSI를 담당하는 주요 기능에 대한 역할
   - Node Plugin Pod : 해당 Pod들이 DaemonSet으로 만들어지고, 각 노드별 해당 솔루션으로서의 기능인 볼륨을 생성하는 역할
   - Longhorn에 대한 Storage Class가 생성
     + 이를 이용해 PVC를 만들면, Provisioning은 PVC를 바라보고 있다가, PVC가 생성된 것을 확인하고 이에 맞는 PV를 만들어서 PVC와 연결
     + csi-resizer : PVC를 모니터링하면서 PVC에 대한 Volume Size가 변하는지 보고 있다가 사용자가 볼륨의 크기를 변경하면 이를 처리
     + csi-snapshotter (Longhorn에는 구현되지 않음) : 볼륨에 대한 스냅샷을 찍음
     + PV가 만들어져 있으면, 특정 노드 위 CSI 플러그인은 해당 노드에 볼륨 솔루션인 매니저를 통해 볼륨을 생성하라고 요청하며, 해당 요청은 최종적으로 엔진까지 가서 엔진이 볼륨 생성
       * Pod를 PVC에 붙이게 되면 Volume Attachment라는 Object가 생성되며, 이를 감시하고 있던 csi-attacher가 볼륨과 Pod를 연결
   - 솔루션 자체적으로 UI가 있는데, 이 UI들은 매니저와 직접 통신하므로 모니터링과 직접 제어가 가능
   - 솔루션 자체적으로 해당 노드에 별도로 설치해야 되는 패키지가 있으면 설치도 해줘야 함
