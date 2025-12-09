-----
### 단계별로 구축해보는 배포 파이프라인
-----
<div align="center">
<img alt="image" src="https://github.com/user-attachments/assets/0d9de4e8-ada7-4064-982f-4ecdfdfb0671" />
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/d31f7c37-4e94-45ee-9c51-1be71044ff49" />
</div>

1. 레벨 1 : Jenkins 기본 구성을 이용해 구축
   - GitHub에서 릴리즈 파일도 가져오며, 실행 순서에 따라 소스 / 컨테이너 빌드 및 배포 진행
   - Jenkins UI를 통해서 생성하므로 파이프라인을 가장 직관적인 형태로 생성 가능

2. 레벨 2 : Jenkins Pipeline 사용
   - 한 구성으로 빌드, 배포가 쭉 연결되므로 한 번만 실행
   - Jenkins Pipeline 사용 이유
     + 시각적인 스테이지 뷰 제공 : 각 단계가 성공으로 잘 끝났는지, 시간은 얼마나 걸렸는지, 클릭을 하면 해당 구간의 로그만 보여주기도 해서 파이프라인 전체가 어떻게 흘러가는지 파악 가능
     + JenkinsFile라는 걸 릴리즈로 관리 가능 : yaml 파일이랑 DockerFile 처럼 사전에 만들어 놓은 스크립트로 Jenkins 파이프라인을 실행시킬 수 있음
     + 코드로 관리함으로써 복사해서 사용도 가능하며 변경 / 이력 관리도 될 수 있음
    
<div align="center">
<img src="https://github.com/user-attachments/assets/19c6c25f-cdcc-446e-b2ad-2c78a5ecdcbb" />
</div>

3. 레벨 3 : Kustomizedhk Helm 배포
   - 레벨 2와 같은 구성에 배포에 kubectl만 Kustomize와 Helm으로 변경
   - 릴리즈 부분도 변경
   - Kustomize와 Helm은 단순 파일을 가져오는 것이 아닌 패키지를 가져오는 것으로, 이 패키지를 사용하면 YAML 파일을 동적으로 구성 가능
     + A라는 App을 만들기 위해 3가지 yaml 파일들을 사용했다고 가정
     + MSA 형태로 구성하여 여러 App들을 구성 : B, C, D와 같은 App들이 생성되는데, A의 YAML 파일들을 복사해서 이름만 변경하는 식으로 작업
     + 각 환경마다 또 환경 정보가 다르게 들어가는 부분이 있으므로, env 부분을 수정
     + 따라서, 코드로 인프라를 생성하는 건 복사를 할 수 있으므로 개수를 늘리는 작업은 매우 쉬움
     + 하지만, 이러한 작업은 매우 귀찮으므로, Service의 옵션 하나만 변경하게 되면 전체 서비스를 건드려야 하므로, Helm Template이라는 것을 만들면, Service의 경우 name이나 select 부분에 static 값이 아닌 변수를 넣게 되고, Helm으로 배포 명령을 전송하면 YAML 파일 생성
     
<div align="center">
<img src="https://github.com/user-attachments/assets/9cb26aa3-9c56-48ae-b4f2-d5bb377579f0" />
</div>

4. 레벨 4 : ArgoCD를 써서 배포를 분리
   - CI / CD 환경에는 빌드까지만 있고, Kubernetes의 인프라 환경에 ArgoCD를 설치 : 내부적으로는 Helm 패키지가 사용되므로 Helm 패키지를 가져와 배포
   - Jenkins 파이프라인을 실행시켜서 소스 빌드와 컨테이너 빌드를 실시 (배포 : 릴리즈 파일을 수정하는 것이 배포 행위)
     + 첫 배포가 된 이후 ArgoCD는 GitHub에 있는 형상관리 파일과 실제 Kubernetes에 배포한 YAML 파일들에 대해 동기화
     + GitHub 패키지가 수정되면 자동으로 Kubernetes의 리소스를 변경해주고, 반대로 Kubenetes 리소스 옵션을 직접 변경하면 GitHub 내용을 업데이트
     + 따라서, ArgoCD가 Kuberenetes와 릴리즈 파일에서 계속 싱크를 맞춰주는 것이 배포가 되는 셈
   - 장점 : 문서와 실체를 항상 똑같이 유지 / Kubernetes 리소스를 편하게 볼 수 있는 UI 제공
   - 또한, Blue / Green이나 Cananry 배포를 쉽게 쓸 수 있는 기능 제공
