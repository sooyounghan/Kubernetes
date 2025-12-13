-----
### Argo Application 설치 및 배포 (kubectl, helm)
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/a02e98a8-cac6-4e52-991d-24f4ada7e541" />
</div>

1. CD, Image Updater, Rollouts 설치 : 모두 Artifcat Hub에서 Helm 패키지로 설치 가능
2. CI / CD Server에 Jenkins로 Argo Application 설치하는 Job을 하나 생성
   - 실행을 하면 GitHub에서 패키지를 다운 받은 다음, Kubernetes 클러스터로 설치
   - ArgoCD가 Kubernetes 클러스터에 잘 설치가 되었다면, Application이 ArgoCD를 통해 배포
     + Git Repository가 존재하며, 배포할 Application에 대한 YAML 파일들이 존재 (kubectl이나 helm, kustomize로 배포 파일들을 올려놓은 곳)
     + Kubernetes 관련 레포지토리를 만들다 보면 통상 3개 생성 : Application 소스 코드 전용 / Application을 배포하기 위한 릴리즈 전용 / ArgoCD와 같은 Kubernetes에 Add-On으로 설치하기 위한 레포지토리로, 각각 분리 해놓는 것이 좋음
       * 각 레포지토리별로 사용하는 담당자 또는 연동 시스템이 다름 (Application 소스 코드는 주로 개발자, Application 릴리즈 전용은 DevOps 엔지니어가 관리, Add-On 설치 전용 레포지토리는 운영자가 통상 관리 및 사용)
       * 상황에 따라 담당할 사람이나 연결을 해야 되는 시스템이 다르므로, 접근 유저별 권한 또는 연동되어야 되는 시스템에 불필요한 코드가 다운되는 것을 방지하기 위함
     + Git에서 Release 파일을 받아서 Kubernetes에 배포하는 것이 최종 목표
       * 하나의 App을 배포하는 단위가 애플리케이션으로, 애플리케이션은 App 별로 여러 개가 만들어질 수 있음
       * Default라는 프로젝트에 소속 : 이 프로젝트는 애플리케이션들을 Grouping하는 용도로 사용
     + Source : Git에 대한 정보를 입력
     + Destination : 배포할 Kubernetes의 클러스터 정보 제공
     + Refresh : ArgoCD를 Git에 연결하면 자동으로 변경 사항이 감지가 되지만, 3분 정도 Check-Interval 존재하는데, 이 버튼을 누르면 바로 변경 사항 확인
     + Synchronization : Kubernetes에 배포를 실행
     + Gerneral : 기본 정보나 배포 시에 줄 옵션
       * Sync Policy : 리소스에 대한 변경 사항이 감지가 됐다면, 수동 또는 자동 배포를 선택하는 옵션 (자동으로 할 경우 3분 이내 배포)
       * Sync Option : 배포 상세 옵션 (예) 배포할 때 네임스페이스를 자동 생성 또는 배포 시 줄 수 있는 기능 존재)
       * Prune Polciy : 리소스 삭제 정책

3. ArgoCD의 세 가지 바식 배포 툴
   - ArgoCD를 쓰면 툴들을 별도로 설치하지 않아도 릴리즈 파일들이 어떤 방식인지에 따라 선택해주면 되는데, ArgoCD가 릴리즈 파일들을 다운받으면 자동으로 어떤 포맷인지 인식
   - Desired Manifest : Git에 있는 YAML 파일을 다운받은 Manifest
   - Live Manifest : Kubernetes에 있는 리소스를 조회한 Manifest
   - 예) Git에 Configmap이 존재한다고 가정 (data: content)
     + ArgoCD는 3분 안에 자동으로 받아오며, Sync 버튼을 누르면 Kubernetes가 배포가 되면서 Live Manifest가 만들어짐
     + Live Manifest는 Kubernetes의 실제 리소스를 그대로 조회하고 있으므로, Kubernetes에서 자원을 직접 수정하게 되면 바로 반영 : 혹은 ArgoCD에서 이 Manifest를 직접 수정해도 Kubernetes의 자원이 바로 수정
     + Desired Manifest는 여기서 직접 수정이 불가 : Git에서 수정을 해야 하고, Refresh를 해야지 변경이 됨
       * 이 상태에서 diff 버튼이 활성화 (Live Manifest의 경우에는 diff 버튼이 활성화되지 않음) : diff의 대상은 Git 변경 사항이 될 Live와 현재 Live임
       * 따라서, Git에도 없는 내용을 새로 추가했다면, diff의 대상으로 나오지 않음
       * 하지만, Git에도 있는 데이터 부분을 Kubernetes에서 수정했다면 Sync를 했다면, Git에 있는 내용으로 다시 변경되므로 diff 버튼이 활성화
       * 즉, Argo CD 입장에서 현재 Kubernetes에 있는 자원과 앞으로 Git에 있는 내용으로 변경될 자원에 대한 diff
      
