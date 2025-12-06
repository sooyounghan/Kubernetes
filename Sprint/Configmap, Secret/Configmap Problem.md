-----
### 영역 파괴의 주범 - Configmap
-----
1. 전체적 배포 흐름과 상황
<div align="center">
<img src="https://github.com/user-attachments/assets/14145117-22ba-4dc8-861d-525b596994db" />
</div>

  - Kubernetes 환경
    + 각 환경마다 Pod 생성되고, Pod에 들어가는 컨테이너는 DockerHub에서 같은 이미지 다운로드
    + 하지만, 환경마다 다른 값을 주려고 Configmap를 별도 설정
    + 따라서, 이미지 안에 변수 값을 받아 실행하는 명령어 존재
    + 개발 환경은 Spring으로 개발하며, Github으로 Commit을 하면, Jenkins에서 이 소스를 받아 파이프라인으로 들어가는 구성
    + 소스 빌드와 컨테이너 빌드 과정에서 컨테이너 이미지가 DockerHub로 올라가게 되고, Container 빌드 후 개발 환경이 바로 이어서 배포
    + QA가 운영은 필요할 때 배포 버튼을 누르게 구성

  - Kubernetes 이전 환경
    + 인프라 담당자가 환경별로 서버 세팅 : OpenJDK도 설치해야 하며, VM 환경 변수 값을 바로 사용할 수 있도록 여러 서버에 똑같은 패턴으로 설치하되 필요 시 다른 값을 주기 위해 환경 변수 별도 관리
    + 배포는 jar 패키지 파일을 VM 환경에 복사해놓고, 실행 명령을 직접 전송하며, DevOps 담당자는 이 App에 환경이나 목적에 맞는 변수를 넣음
    + 환경별로 Properties 파일을 관리
    + 즉, 각자 영역에서 담당자들이 관리하는 환경 변수가 존재 (이를 Kuberentes에서는 Configmap이 사용)

2. Configmap은 목적만 보면 간단해보일 수 있지만, 프로젝트 상황에 따라 영역을 넘나들 수 있고, 정답이 없어 영역파괴의 주범이 될 수 있음
   - 기능적인 부분만 생각하면 Properties 값을 Configmap으로 설정해도 되지만, 이는 관리 방법에 따라 다르므로, 문제 발생 가능
