-----
### Argo CD Architecture
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/159f3445-61cd-45d8-b416-7274eb3c126f" />
</div>

1. Kuberentes 전용 배포 툴로이며, 릴리즈 파일 저장소(변경 관리 저장소)로 Git을 반드시 필요로 함
2. Image Updater : AgroCD의 추가 기능
   - DockerHub에서 컨테이너 이미지에 대한 변경을 감지해 배포를 할 수 있는 파이프라인을 만들 수 있음
3. 고급 배포를 지원해주는 Rollouts : 특정 배포 전략으로 Kubernetes 자원을 생성
4. Argo Events는 Kafka와 같은 역할을 하는 솔루션 (이벤트 버스 구조 아키텍쳐)
   - 시스템들 간 이벤트를 주고받는 메인 통로 역할
   - 이 중, Workflow로 보내지는 이벤트가 있을 수 있고, Workflow 안에는 받은 이벤트 내부 값에 따라 어떤 작업을 실행하라는 순서도가 존재
   - 해당 Workflow 결과에 따라 작업이 실행 되는데, 그 중 배포하라는 작업이 존재할 수 있어 CD가 실행
5. Argo Workflow는 Airflow, Cubeflow와 같은 역할을 하는 Workflow Managment 도구
6. ArgoCD가 Kubernetes에 설치됐을 때 아키텍쳐
   - Kubernetes 설치 : 30000번 Port를 통해 UI에 접근하거나 kubectl로 CLI를 전송 - kube-apiserver를 통해 API를 받아 관련된 컴포넌트들에게 트래픽을 전달해주는 구조
   - ArgoCD를 설치하면 수 많은 컴포넌트들이 설치
     + Server : API 서버와 대시보드 역할을 동시에 하므로, NodePort로 ArgoCD UI에 접속할 수 있고, kubectl처럼 argocd라는 툴을 설치해 CLI을 전송 가능
     + GitHub (App에 대해 배포할 YAML 파일들이 존재)가 존재하며, Repo Server는 Git에 연결해서 YAML 파일들을 가져오며, 이를 통해 Kubernetes에 배포할 Manifest 파일을 만들어 놓는 역할을 함
     + Application Controller는 Kubernetes의 리소스를 모니터링하면서 Git에 받은 내용과 다른게 있는지 비교하며, 내용이 다르면 Git에 있는 내용으로 배포 진행
     + Kuber API : Kubernetes로 리소스 생성 명령을 전송하는 역할
     + Notification : ArgoCD에서 발생하는 이벤트를 외부로 Trigger해주는 역할
     + Dex : 외부 인증 관리를 하는 역할로, Grafana와 같은 대시보드들에 대해 IAM 솔루션을 사용하여 사용자가 로그인을 하면, IAM 솔루션과 연결된 시스템에는 별도로 로그인을 하지 않아도 자동 로그인이 되도록 설정 (Single Sign-On) / Kubernetes에는 대표적으로 Key Clock이 존재
     + Redis : 메모리 DB (시스템에서 캐시 역할을 하며, GitHub와 연동되는 구간이랑 kube-apiserver서버랑 연동되는 구간에 캐시로 사용되어 불필요한 통신을 줄여줌)
     + ApplicationSet Controller : Multi Cluster를 위한 App 패키징 관리 역할 (ArgoCD는 클러스터마다 설치할 수 있지만, 하나만 설치해서 여러 클러스터로 App 배포 가능한데, 이러한 상태일 때 배포환경 마다 ArgoCD에서 배포 구성에 대해 환경별로 다른 부분만 셋팅해서 사용할 수 있는 템플릿 제공)
