-----
### ArgoCD Image Updater를 사용한 이미지 자동배포
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/97f976c3-1bd5-44ae-8b8a-0d6652211ee2" />
</div>

1. 사용 이유
   - 개발자 PC에서 개발자는 자신의 App 코드를 App 소스 코드 전용 레포지토리에 Commit
   - DevOps 엔지니어는 이 App에 대해 Kubernetes에 배포할 YAML 파일들을 릴리즈 전용 레파지토리에 COMMIT
   - 이런 상태에서 배포해야 되는 상황
     + Resource 스펙을 변경해야 될 때 : Deployment에서 배포 전략을 바꾼다던지, 수동으로 Scale-Up해야 할 때, DevOps 엔지니어는 YAML 파일을 수정해서 Git에 커밋하고 Deployment Job을 실행하면 Kubernetes에 반영
     + App 버전이 업그레이드 되서 컨테이너 이미지를 변경해줘야 할 때 : 개발자가 소스 빌드 Job을 실행하면, 최신 소스를 가지고 Jar 파일이 만들어지고 컨테이너를 빌드를 하며 DockerHub에 이미지를 업로드
       * 이어서 배포가 실행되는데, 그 전에 DevOps 엔지니어가 YAML 파일에서 이미지 태그를 수정하고 Commit을 해줘야 되지만, Helm을 사용하면 별도 수정 없이 배포 가능
     + 즉, 배포를 해야 되는 상황 2가지가 존재하는데, DevOps 엔지니어 입장에서 Resource 스펙이 변경될 때는 YAML 파일을 수정 하고 배포를 실행하는 작업을 해야지만 App 버전이 업그레이가 되서 소스 빌드를 실행할 때 뱊포 부분을 자동화시켜 놓을 수 있음

2. Argo CD 배포
   - Resource 스펙이 변경이 될 때 : Argo CD에서 변경 감지를 해주므로 Kubernetes에 자동 배포
   - App 버전 업그레이드 : 배포 자체를 Argo CD에서 진행하므로 컨테이너 빌드가 끝난 이후 자동 배포가 어려움
     + 따라서, Jenkins에서 YAML 파일을 수정하고 Git에 업데이트를 시켜주는 스크립트를 생성시켜주기도 함 (하지만 이 방법은 로직이 복잡)
     + 이를 해결하기 위한 것이 Argo CD Image Updater

3. Argo CD Image Updater
   - DockerHub를 계속 모니터링하면서 이미지 업데이트가 감지가 되면 ArgoCD에 배포를 하라는 명령 전송 (단, 배포 패키지를 Helm과 Kustomize를 사용했을 때만 가능 : 내부적으로 --set image.tag=... 명령어를 사용하기 때문임)
   - Image Updater를 설치할 때, DockerHubdhk Argo CD에 대한 연결 정보를 입력해야 되며, Argo CD의 각 App별로 업데이트를 어떻게 할 것인지에 대한 태그 규칙이 추가하는 곳이 존재
