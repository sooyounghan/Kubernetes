-----
### DevOps 전체 구성도
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/b03f3abd-0763-463f-a51b-376d5e112386" />
</div>

1. 개발 소스를 Github로 Commit 하면서 소스 코드를 통합적 관리를 하다가 CI / CD 환경에서 Build 버튼을 누르면, Github에서 최신 소스 코드 다운
2. 본격적으로 Gradle 소스 빌드가 시작 되면서, Maven 저장소에서 소스에 필요한 라이브러리들을 다운
3. 실행할 수 있는 jar 파일이 만들어지면서 소스 빌드 종료
4. Kubernetes 환경으로 배포해야 하므로, Container 빌드를 한 번 더 실시
   - Docker로 빌드가 시작되면서 DockerHub에 OpenJDK가 있는 베이스 이미지 다운
   - 받은 이미지에 내 파일을 넣으면 App이 컨테이너 이미지로 생성되면서 DockerHub에 적재
5. 배포는 kubectl 명령어를 통해 Kubernetes에 Pod를 생성시키면 Kubernetes가 필요한 이미지를 DockerHub에서 다운받고 containerd한테 컨테이너 생성 요청
