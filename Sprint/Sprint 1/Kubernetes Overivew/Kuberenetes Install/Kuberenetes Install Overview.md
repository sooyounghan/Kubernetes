-----
### 쿠버네티스 무게감 있게 설치하기
-----
1. 전체 환경
<div align="center">
<img src="https://github.com/user-attachments/assets/a285c116-d2d0-4372-8488-229e6909888c">
</div>

2. 개발 환경
   - 개발 툴 : IntelliJ
   - 개발 언어 / 패키지 : JAVA, OpenJDK
   - 개발 프레임워크 : Spring Boot Framework

3. CI / CD 환경
   - Virtual Box
   - Guest OS : Rocky Linux
   - 빌드 / 배포 도구 : Jenkins
   - 컨테이너 빌드 : Docker

4. 인프라 환경
   - Virtual Box : Rocky Linux 설치
   - 컨테이너 오케스트레이션 : Kubernetes
   - 컨테이너 런타임 : containerd

5. 전체 흐름
   - 개발 환경에서 개발 툴로 IntelliJ를 이용해 소스 코딩 (Spring Boot를 이용해 개발 편의 제공 활용)
     + Maven이라는 Repository에서 라이브러리를 다운해 Gradle로 가져옴
     + Gradle을 통한 빌드 : 소스 코드를 컴파일하여 패키지로 실행할 수 있는 패키지(jar) 생성
     + 해당 파일을 사전에 설치된 JVM 위에서 실행

   - 개발이 완료되어 소스 코드를 Commit하면, Github로 코드 통합
     + Jenkis로 빌드 : Github를 통해 소스 코드를 다운 받아, Gradle에 있어 필요한 라이브러리들을 다운로드하고 jar 파일 생성
     + 컨테이너 빌드(jar 파일 이미지 생성) 과정 : 컨테이너 이미지 저장소인 DockerHub를 통해 OpenJDK 베이스 이미지(App을 실행하기 위한 기반이 되는 환경)를 다운
     + 다시, DockerHub에 이미지 업로드

   - 배포 과정
     + Jenkis에서 Pod 생성 명령을 Kubernetes에게 전송
     + Pod 안에는 컨테이너 이미지 주소가 존재하여, Kuberentes는 이 주소를 보고 DockerHub에서 컨테이너 이미지를 다운로드
     + 컨테이너를 그 이미지로 생성해달라고 요청 (컨테이너 런타임 활용)

6. Kubernetes 구축 범위
   - 인프라 환경 구축 후, 완료되면 PC에 브라우저를 띄우고 Kubernetes 대시보드에 접속
   - 이후, Kubernetes Object 생성 (이 때, 사용되는 컨테이너 이미지는 DockerHub 업로드 이미지 활용)
   - 원격 접속 툴(MobaXterm)을 설치해 Linux로 접속하면 kubectl이라는 툴이 설치되어 있고, 이를 통해 Kubernetes에 CLI 통신 명령 전송
