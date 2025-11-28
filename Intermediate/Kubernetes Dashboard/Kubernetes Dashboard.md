-----
### Kubernetes Dashboard
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/12c7228e-4ac4-4cb2-bc74-4fdaf88532f8">
</div>

1. Dashboard가 설치되어 있는 상태에 대해 분석 및 보안에 적합하도록 설치
   - Namespace에 deploy Pod가 설치되어있으며, Service와도 연결되어 있음
   - 사용자는 보안 없이 HTTP API로 대시보드에 접근 가능
   - 대시보드 화면이 열리고, Skip 버튼을 누른 뒤, 별도 인증 없이 로그인해서 대시보드를 통해 Kubernetes Object를 조회하고 생성 가능 : Pod가 API 서버에 접근할 때부터 클러스터 자원에 대한 권한을 가지고 있기 때문에 가능

2. Pod가 Cluster 자원을 접근할 수 있는 이
   - Pod는 Service Account가 연결이 되어 있고, 또 Service Account에는 Role Binding을 통해 Role이 연결 : Role에 대한 권한은 Namespace에서만 사용할 수 있으므로 이 상태에서는 Pod가 모든 자원에 접근 불가
     + Kubernetes를 설치하면 기본적으로 만들어지는 Role 중 Cluster Admin이라는 Cluster Role이 존재
     + Cluster Role Binding을 하나 새로 생성해 Cluster Admin과 Service Account를 연결하게 되면, Dashboard는 Admin 권한으로 API 서버 접근 가능
     + 이러한 설정이 없다면 Dashboard는 선정된 자원에만 접근

3. ReplicaSet의 경우 서버의 IP와 Port 번호만 알면 Cluster에 접근 가능하므로 위험함
4. 보완적인 접근 방법
   - API 서버로 바로 접근을 하기 위해 Kubernetes의 kubectl 파일에 클라이언트 Key, 인증서가 필요 : 따라서, 두 파일을 합쳐서 clientp12 파일을 생성하고, PC에 인증서 등록
   - HTTPS로 사용자는 API 서버를 통해 대시보드에 접근 가능하며, 대시보드는 사용자가 필요한 자원을 조회하거나 만들 때, API 서버로 해당 요청을 하ㅔ게 됨
   - 따라서, Service Account에 연결된 Service Account 값으로 대시보드에서 Token Login이 가능
     + 다른 사람의 입장에서는 IP와 Port 정보만으로 접근 불가 : Kubernetes 인증서 및 대시보드에 접근되더라도 로그인을 할 수 있는 토큰 값을 알아야 함
   - 따라서, 보안이 중요한 환경에서는 대시보드 설치가 필
