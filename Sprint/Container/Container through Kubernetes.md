-----
### Kubernetes 흐름으로 이해하는 컨테이너
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/3a42295a-25ed-45f7-8531-7ff3001bc59b">
</div>

1. Container의 Kernel : chroot, cgroup, namespace (커널 레벨 기술)
2. Container Runtime : 컨테이너를 생성해주는 역할
   - High Level
     + Docker : libcontainer 기반으로 만든 사용자 친화적 컨테이너 런타임 (애플리케이션을 독립적인 환경에서 띄울려고 사용)
   - Low Level
     + LXC : 커널 레벨의 기술을 가지고 만든 컨테이너 런타임 (운영체제를 컨테이너 가상화로 나누기 위한 목적)
     + libcontainer : Docker의 기반이 되는 컨테이너 런타임
     + rkt 

3. Container Orchestration (Kubernetes Runtime)
   - Kubernetes에 컨테이너를 하나 생성하는 명령은 없고, Pod를 생성하는 명령이 존재하는데 이 Pod 안에 Container를 하나 이상 명시 가능
   - Component
     + 예) Pod 생성 명령 전송
     + kube-apiserver : Kubernetes로 보내지는 모든 API를 받음
     + kubelet : Pod 생성을 담당하는 kubelet에게 전달 (만약, Container가 2개라면, 이를 생성하도록 Container Runtime에게 전송하며, 이를 토대로 두 개의 컨테이너를 생성)
       * Container Runtime이 받을 수 있는 형태의 API를 호출
       * CRI의 내부 간 통신은 grpc로 이루어짐
       * 컨테이너 런타임 기술 개발과 Kubernetes 기능 개발은 서로 별개의 영역이지만, Docker에 새 기능이 추가되면 Kubernetes도 같이 패치를 해야 하는데, 즉, 구조상 컨테이너 런타임이 변경될 때마다 CRI 구현체도 변경해야 하므로 Kubernetes도 패치가 되어야 하는데, 이를 위해 kublet에서 Container Rumtime을 바로 받을 수 있도록 구조를 변경 : containered에서 CRI-Plugin이라는 기능 추가
       * cri-o(runC 사용)는 태생부터 RedHat이 이 규격을 맞춰 만든 런타임이며, 미란티스 컨테이너 런타임도 동일
       * 따라서, Kubernetes에서는 1.24버전 부터 Pod를 생성하려 할 때, 기본적으로 위 구조를 기본 동작으로 실행

    - 시간이 지나면서, Docker에서 containerd가 따로 분리가 되서 나오게 되었음

4. Kubernetes는 1.5버전부터 Interface를 추가
   - 구현부를 따로 추출 (Container Runtime Interface)라고 해서 컨테이너 런타임 인터페이스라고 부름 : kubelet에 인터페이스 규격을 정하고 규격에 맞게 구현부 정의하며, 구현부에서 각 컨테이너 런타임을 호출)
   - 구현부는 Kubernetes Project에 위치 : 각 컨테이너 런타임 측에서 Kubernetes 프로젝트의 소스를 Contribute하는 형태 

5. Docker가 cri-dockerd라는 어댑터를 제작 : docker를 다시 지원할 수 있게 되는데, 이를 미란티스 컨테이너 런타임이라고 부름

6. OCI (Open Container Initiative) : 컨테이너 런타임이 컨테이너를 만들 때, 지켜야되는 표준 규약들을 관리 (따라서, 런타임끼리 공유 가능)
   - Docker는 OCI 규격을 맞추기 위해 Low-Level에 runC를 만들엇고, containerd에서도 이를 쓰도록 변경
   - rkt도 OCI를 맞추므로 공유 가능
   
8. Docker Engine
<div align="center">
<img src="https://github.com/user-attachments/assets/12de043b-4e7b-4837-8156-225d043cf031">
</div>

   - dockerd 라는 컨테이너를 만들어주는 기능 외에도 CLI 제공(CLI/API), Log 관리 기능(Logs), Storage Driver를 제공(Storage), Network 생성(Network)
   - 컨테이너를 만들어주는 역할 : containerd / libcontainer : containerd가 Low Level인 libcontainer를 이용
