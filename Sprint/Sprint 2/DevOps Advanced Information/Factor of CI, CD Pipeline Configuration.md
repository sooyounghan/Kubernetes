-----
### CI / CD 파이프라인을 구성할 때 고려해야 하는 요소
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/0743ae56-dbb4-4cac-b808-a3f56e8152f2" />
</div>

1. CI / CD 환경, 인프라 환경이 존재
   - Jenkins를 통해 파이프라인 구축 : 순서대로 파이프라인을 만들고, 각 실행했던 방식과 달리 하나의 구성 안에 파이프라인이 자동으로 연결되는 형태로 생성
   - 배포는 kubectl 배포 외에 Helm, Kustomize라는 것으로도 배포 가능 

2. Jenkins에서 소스 빌드랑 컨태이너 빌드만 실시
   - Argo CD라는 Kubernetes 전용 배포 툴을 이용하면, 각 Kubernetes 환경에 배포를 할 수 있음
     + 배포 툴은 Jenkins가 아닌 Argo CD를 통해서 사용 : 배포 영역을 빌드와 구분하기 위해 많이 사용
     + 배포와 인프라 환경의 관계를 1:다로 구성 가능 (하나만 관리해서 편의성은 좋지만 개발 환경 때문에 장애가 발생하면 운영 환경에도 배포를 못하게 되는 문제 발생) 또는 Argo CD로 1:1 구축 가능 (단점 : Argo CD를 이중 관리해야 함)

3. CI / CD 툴
   - 온라인용으로 Github Actions, 오프라인 용으로는 Jenkins, JenkisX, TEKTON이 존재
     + 온라인은 인터넷 환경에서 사용하므로 CI / CD 서버를 별도로 만들 필요가 없음
     + 오프라인에서는 Jenkins X는 Jenkins에서 컨테이너 환경에 맞춰 새롭게 만든 것
     + TEKTON : 컨테이너 환경에 최적화 된 CI / CD를 목적으로 만들어진 도구
   - CI / CD 서버는 개발 소스나 릴리즈 파일들에서 중요한 정보가 많으므로, 오프라인 툴을 쓸 수 밖에 없음
   - 컨테이너 환경 툴 비교
<div align="center">
<img src="https://github.com/user-attachments/assets/d4841f76-a70f-41eb-a86f-d45f2cfe9c41" />
</div>

4. 즉, 온라인과 오프라인 제품의 선택 그리고 레퍼런스가 많은 제품인지 보는 게 좋음
   - 또한, 제품을 도입하고 계속 유지보수로 계약할 수 있는 업체가 있는지의 유무도 중요

5. Docker 대체
   - Docker의 무거움과 Daemon(Linux의 백그라운드로 항상 실행되고 있어야 하는 실행 가능판 제품)이 필요
   - 컨테이너 빌드로 Builder가 존재 : Podman을 이용해 도커 로그인을 하고, 이미지를 가져오는 기능도 있어서 OpenJDK 같은 베이스 이미지를 가져오며, 컨테이너 빌드를 하고 이미지가 만들어지면, skopeo가 이미지 푸시 기능이 있어 DockerUp으로 이미지를 업로드
   - ArgoCD를 통해 Kubernetes 배포가 되는 구성
   - 도커보다 자원 사용률은 낮음
   - Redhat에서 밀고 있으므로, OpenShift와 같은 관리형 Kubernetes 제품에 기본적으로 포함

6. Kaniko : Kubernete 위에서만 돌아가는 제품
   - 컨테이너 환경에서 동작
