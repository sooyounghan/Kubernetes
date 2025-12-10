-----
### Helm, Kustomize 최종 비교
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/269248ab-7f4b-4ceb-9f83-99b2a0402322" />
</div>

1. 공통점
   - 사용 목적
     + 배포를 하기 위해 만들어야 하는 YAML 파일들에 대해 중복 관리 최소화를 위해 사용
     + MSA를 사용이 증가하면서, App을 잘게 쪼개게 되고, App 종류가 다양해짐

   - 두 패키지 매니저는 다양한 배포 툴에서 지원
     + 즉, Helm이나 Kustomizer를 더 편하게 쓸 수 있도록 부가적 기능들을 제공
     + 반대로, 이 두 패키지 매니저가 거의 대표적이므로 두 패키지 매니저를 잘 지원해줘야 배포 툴을 선택하는 요인이 됨
    
2. 차이점
   - 배포 편의 기능 : Helm은 200개 정도 되지만, Kustomize는 10개 정도 구성 (상대적 비교)

3. 각 패키지 매니저를 이용해 하나의 패키지를 만들었을 때, 그 패키지를 활용할 수 있는 법
   - Helm은 한 패키지를 만들어서 MSA 목적으로 다양한 환경으로 배포하는데 사용해도 무방 (패키지를 구성할 때, 두 목적을 챙기면서 만들 수 있음)
   - Kustomize는 둘 중 한 목적을 선택해서 패키지를 구성하는 것이 좋음 : 활용 범위를 늘릴 수록 점점 패키지 내부 파일 양이 많아지는 구조임

4. 사용 목적
   - Helm : 기업 제품을 패키지하는데에도 사용 (자신의 환경에 따라 조절을 할 수 있도록 패키지 생성 가능)
   - Kusotmize : 프로젝트 App을 패키지화시켜서 배포하는 목적

6. Use Case
   - Helm은 App이 5개 이상 있고, 주로 대형 프로젝트할 때 많이 사용
   - Kustomize는 App 종류가 5개 미만의 간단한 프로젝트를 할 때 주로 사용
   - Kubernetes를 사용할 때, 여러 오픈소스를 도입하게 되고, 오픈소스는 대부분 Helm 형태로 배포
   - Kustomize를 알아야 할 경우가 있다면, 처음부터 끝까지 소규모 프로젝트 또는 배포 파이프라인을 빠르게 구축
     + Helm은 단계적으로 적용해도 되는 경우

7. 설치 구성
<div align="center">
<img src="https://github.com/user-attachments/assets/5657c576-8e3a-41cc-b1e7-c78fd6ca713d" />
</div>

  - Kustomize
    + 자체적으로 Kustomize를 관리하는 사이트가 있으며, 다운 받거나 릴리즈를 관리하는 GitHub 존재
    + 1.14 버전부터는 kubectl에 통합되어 현재 버전인 1.27에는 Kustomize가 포함되어 있어 바로 사용 가능
    + 따라서, kubectl 명령에 -k가 Customize를 사용한다는 의미
    
  - Helm
    + 자체적으로 Helm 사이트가 있고, GitHub에서 Helm을 다운받아 설치
    + kubectl과 동일하게 인증서가 있어야 kubectl 서버로 통신이 가능
    + 설치가 되면 helm으로 시작하는 명령어를 전송해 API가 전송됨

8. Helm : Artifact HUB (Helm 패키지 저장소) :  GitHub나 DockerHub와 같은 역할이라고 보면 되며, 많은 오픈소스 기업들에서 자신의 제품들을 Helm 패키지로 설치할 수 있도록 패키지 공개
9. 사용 방식의 차이
<div align="center">
<img src="https://github.com/user-attachments/assets/09624e59-732a-42a0-bfb0-a3e4e93c4514" />
</div>

   - Helm : 함수 방식의 컨셉으로 패키지 사용
     + template이라는 폴더 안에 배포해야 될 YAML 파일들이 존재
     + helm 명령어 존재 : 파라미터를 주입하며, 파라미터에 변수를 넣어서 YAML 파일 생성하여 Kubernetes API로 전송
     + 즉, 함수 방식은 template이 존재하고, Input Parameter에 내부 로직에 따라 값이 치환이 되서 결과 값이 만들어짐

   - Kustomize : Overlay 컨셉으로 패키지 사용
     + base 폴더 안에 아무런 처리가 되지 않은 YAML 파일 존재 : 이대로 배포도 할 수 있으며, 명령을 통해 배포 가능
     + kubectl apply -k : Kustomize 패키지로 배포한다는 의미
       * 기존 파일 : 개발 환경에 배포를 할 때 달라지는 스펙을 폴더에 따로 명시해 놓은 YAML 파일 예 (배포를하게 되며, TargetPort 값이 없으므로 에러 발생)
       * kubectl apply -k ./app/overlays/dev : base 파일에 YAML 파일을 재구성 (기본적인 값은 그대로 반영, 같은 내용이면 Overlay한 dev로 값을 수정함 : 따라서, NodePort처럼 없는 내용이면 추가)

   - 즉, Helm 패키지는 배포할 대상이 늘어날수록 명령어가 변경되는 반면에, kUSTOMIZE는 명령어는 단순한데 파일들이 늘어나는 구조이므로 사용 목적이 많아질수록 관리가 복잡해질 수 밖에 없음
