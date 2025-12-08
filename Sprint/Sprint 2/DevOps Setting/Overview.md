-----
### 손쉽게 DevOps 환경을 구축하는 방법
-----
1. 실습 환경
<div align="center">
<img src="https://github.com/user-attachments/assets/66d3e8f2-1c16-4238-823b-dac8858a9593" />
</div>

2. Sprint 2 환경
<div align="center">
<img src="https://github.com/user-attachments/assets/c4db0b6d-6c13-40ee-b9cd-01a498537b60" />
</div>

   - VirtualBox랑 Vagrant를 이용하면 GutesOS가 생성되고, 스크립트를 통해 프로그램들이 설치
   - 브라우저에서 JenKins에 접속하여 빌드 진행 : 빌드를 실행하면 GitHub에서 소스를 다운받아 빌드가 실행
   - 컨테이너 빌드를 하면 소스 빌드를 해서 만들어진 jar 파일이 사용이 되어 Container 이미지가 만들어지고, DockerHub로 업로드
   - 컨테이너 빌드나 배포를 하면, GitHub에서 릴리즈 파일들을 다운 받아 배포 실시 : 배포되는 yaml 파일에 들어가는 이미지 명에 본인의 DockerHub 사용자 이름이 있어야 이미지를 가져올 수 있음

3. VagrantFile로 설치되는 내용
<div align="center">
<img src="https://github.com/user-attachments/assets/4fb2c8dd-6070-4c06-9e62-64cc82e6469e" />
</div>

   - 자원 할당을 보면 CPU 2Core에 메모리는 2Gi, 디스크는 30Gi를 설정
   - 네트워크 설정은 IP는 인프라 환경이 30번, CI/CD 환경은 20번
     + Host-Only Network는 VM 간 통신을 하거나 호스트 PC에서 VM을 접속하기 위한 네트워크로 IP가 동일하면 안 되며, 외부 인터넷에 접속은 안 됨
     + 따라서, NAT를 추가하여 IP는 자동 할당 되어 IP가 인프라 환경과 같지만, 인터넷만 쓰기 위한 용도
    
   - 설치 스크립트 부분
     + 리눅스 기본 설정
     + kubectl 설치 : Jenkins 배포 시에 사용
     + NAT를 설정 : 외부 저장소에서 kubectl 패키지를 다운 받아 설치 가능
     + Docker 설치 : Jenkins에서 컨테이너 빌드 및 DockerHub로 이미지를 올림
     + OpenJDK, Gradle : 소스 빌드
     + Git 설치 : 빌드에 쓸 소스 코드와 릴리즈 파일을 가져옴
     + Jenkins : OpenJDK 11버전을 하나 더 설치 (Jenkins는 Version 11에서 동작하므로 Jenkins 설치용으로 필요 / 소스코드는 17버전으로 동작)

4. Sprint 2 범위 실습 환경
<div align="cetner">
<img src="https://github.com/user-attachments/assets/0429fc94-17a4-4db4-9e83-8a886fda761a" />
</div>

   - Vagrant로 CI / CD 서버가 생성되면, 원격 접속 툴로 접속
   - Jenkins Dashboard에도 접속할 수 있는데, 접속을 하면 최초 Jenkins 초기 셋팅을 한 번 더 하게 됨
   - 전역설정을 통해 직접 설치한 버전의 Gradle과 OpenJDK를 Jenkins에서 빌드를 할 때 사용하겠다고 등록
   - DokcerHub 가입
   - DokcerHub 사용 설정
     + CI / CD 서버 내 도커 허브로 이미지를 올릴 수 있도록 로그인
     + Jenkins에서 Docker를 사용할 수 있도록 권한 부여
   - 인프라 환경에서 인증서를 CI / CD 환경으로 복사 : Jenkins에서 kubectl로 배포할 때 Kubernetes에 API를 전송 가능
   - GitHub 가입 및 설정 : Dockerfile과 yaml 파일 이미지 부분에 내용 수정
   - Jenkins에서 빌드 / 배포하기 위한 프로젝트 설정 및 실행 : Kubernetes에서 리소스를 생성, 그 리소스들 중에서 Pod를 만들 때 이미지는 본인의 DockerHub에 가져감
     
