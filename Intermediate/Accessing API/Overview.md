-----
### Accessing API
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/25e91154-2567-42d0-8089-9cffaadaeda4">
</div>

1. Master Node에 Kubernetes API 서버가 있는데, 이 API 서버를 통해서만 Kubernetes의 자원을 만들거나 조회 가능
   - Kubernetes를 설치할 때 kubectl도 설치해서 CLI로 자원을 조회할 수 있는 이유도 API 서버에 접근을 해서 정보를 가져오는 것
   - 내부에서는 그림과 같이 접근할 수 있지만, 외부에서 이 API 서버로 접근하려면 인증서를 가지고 있는 사람만 HTTPS로 보안 접근이 가능
   - 하지만, 만약 내부 관리자가 kubectl 명령으로 Proxy를 열어줬다면 인증서 없이 HTTP로 API 서버 접근 가능

2. kubectl은 Master 내부에만 설치할 수 있는 것이 아닌 외부 PC에서도 설치하여 사용 가능
   - Config 기능을 활용하면 Kuberentes Cluster가 여러 대가 있을 때, 간편한 명령을 통해 접근할 수 있는 클러스터의 연결 상태를 유지할 수 있음
   - 연결된 상태에서는 kubectl get 명령으로 해당 클러스터에 있는 Pod 정보를 가져올 수 있음
   - 이러한 접근 방법은 유저 입장에서 API 서버에 접근하는 방법으로, User Account라고 함

3. Pod 입장에서는 API에 마음대로 접속하게 되면, 누구나 Pod를 통해 API 서버에 접근하므로 보안상 문제 발생 가능
   - 따라서, Kubernetes에서는 서비스 계정이라고 해서 Pod가 API 서버로 접근하는 방법 존재
   - 결국 Kubernetes에는 유저들이 API 서버에 접근하기 위한 User 계정과 Pod들이 접근하기 위한 Service Account로 분류

4. Authentication : Kubernetes API에 접근
   - 접근을 한 다음, 필요로 하는 자원을 조회할 수 있는 권한
   - 만약, NameSpace로 Pod가 분리되어 있는 상태에서 NameSpace에 있는 Pod가 API 서버에 접근을 할 수 있다고 해서, A NameSpace에 있는 Pod를 조회를 하는 것은 권한 여부에 따라 가능하게 할 수도 있고, 못하게 할 수 있음
   - 권한까지 문제가 없다면 마지막으로 Admission Control이라고 해서, PV를 만들 때, 관리자가 용량을 1GB 이상 만들지 못하도록 설정을 했다면, Pod를 만들라는 API 요청이 들어왔을 때 Kubernetes는 설정된 크기를 넘지 못하도록 해야함
