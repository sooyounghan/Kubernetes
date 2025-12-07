-----
### 전체 개요
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/b33b47db-598b-4424-9d80-da89dfa5870c" />
</div>

1. Kubernetes 구축 : VM에 Master Node를 생성
   - kubeadm 명령 : Cluster 생성 (/etc/kubernetes/manifest 경로에 Pod를 생성시키는 yaml 파일들을 통해 Component 생성하는데 이를 Control Plane Component)
   - 다른 VM으로 여러 대의 Worker Node를 생성할 수 있고, 같은 내용들이 설치되며, Worker Node와 Master Node를 조인시키면 Worker Node 컴포넌트 영역이 생성
     + Kubernetes는 kube-proxy라는 컴포넌트만 추가로 생성 : 사용자가 App을 올릭 ㅣ위한 공간
     + 애플리케이션은 Worker Node에 올리고, 자원이 더 필요한 만큼 Worker Node를 추가
   - Dashboard와 Metrics 서버 같은 App들은 Kubernetes의 기본 기능을 확장시키기 위한 App들을 Addon Pod
   - Kubernetes 인증서 (/root/.kube/config)가 kubectl에 존재하며, kubectl이 kube-apiserver로 API를 전송해 etcd에 데이터를 저장했던 것

2. Controller, Object
   - Object : 각 Object 하나가 인프라 개념으로서 단독 기능
   - Controller : 타 Controller나 Object를 제어하는 기능
   - 이 전체를 칭할 때는 Resource라고 부르며, Resource는 Namespace Level Resource와 Cluster Level Resource로 나눠짐
   - 인프라에는 네트워크나 볼륨, 환경 변수라는 요소들이 존재하는데, Kubernetes는 이 요소들을 Object로 정의하고, Object들을 자동화시킬 목적으로 Controller 개념 생성
     + 이 전체 리소스들은 Control Plane Component라는 Pod들에 의해 동작
     + 각자의 역할이 존재하며, 리소스들을 동작시키기 위해 타 Component들과 통신도 하지만, 실제 컨테이너는 containerd가 생성해주는데, Kubernetes는 Container 생성 요청만 함
     + 
