-----
### DevOps를 구성하는 오픈 소스들
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/4ca806bb-d258-4437-b7a9-095bf9f72b11" />
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/8edd0d33-cf89-4581-8151-7f47b0b63e07" />
</div>

1. DevOps(Development, Operations)가 개발에서 운영까지 원할한 흐름을 만드는데 가장 중요한 역할 : CI / CD
   - CI (Continuous Integrity, 빌드 / 테스트 자동화) : 통합된 소스를 가지고 빌드 테스트를 자동화 시키는 기능
   - CD (Continuous Deployment, Delivery / 배포 자동화) : 배포를 자동화시키는 기능
   - 즉, 개발을 해서 Commit을 하는 순간 운영 환경에서 App이 자동으로 배포되는 파이프라인이 만들어지는 것

2. 8단계 세부 단계 : 계획 → 개발 → 빌드 → 테스트 → 릴리즈 → 배포 → 운영 → 모니터링
   - 계획 단계
     + 일정, 이슈 : REDMINE, Jira, Notion을 사용 (개발 일정을 공유하고 이슈 사항을 기록하기 위함)
     + 협업 : 서로 메신저를 통해 개발과 관련된 대화로, Slack 사용
     + 외부에 위치 (기업은 외부용은 잘 사용하지 않으므로, GitLab이나 Docker Registry과 같은 Private하게 서버에다가 설치를 할 수 있는 내부용 사용)

   - 개발 단계
     + Junit : 테스트 코드를 작성할 때 필요
     + FindBug, PMD : 코드 분석 (코드 패턴에 버그가 있는지 확인)
     + 개발을 서포트하는 다양한 툴들이 있음
    
   - 빌드 단계
     + 빌드 대상은 소스와 컨테이너
     + Gradle을 일반적 사용 : 하지만, Maven 저장소에서 라이브러리를 가져오는 것
     + Public 용으로 Maven을 사용, Private 용으로는 nexus repository 사용

   - 테스트 단계
     + 테스트 요소 : 기능, 성능, 커버리지
     + 각자 개발 단계에서 테스트 코드를 만들고 실행했더라도 코드들이 병합된 후 다른 결과가 나올 수 있으므로, 빌드 단계에서 JUnit을 실행시켜 자동으로 테스트를 한 번 더 실행
     + 테스트 이후 JACOCO라는 툴을 사용해 테스트 시 사용된 로직들이 App 전체 어느 정도 범위를 테스트를 해볼 것인지 커버리지 결과를 알려줌
     + JMeter : 성능 테스트 툴 (자동화 기능이 아닌 개발 환경이나 검증 환경 대상으로 진행)

   - 릴리즈 단계
     + 배포 가능한 패키지를 만드는 과정
     + Dokcer 빌드를 위해서는 DockerFile이라는 스크립트 작성 필요
     + Kubernetes 배포를 위해 yaml 파일을 사전에 작성해야 됨
     + 이러한 배포를 위한 별도의 패키지를 만드는 것
     + 해당 파일들도 변경 관리가 되어야 하므로 작성한 내용을 GitHib에 Commit

   - 배포 단계
     + 배포흘 하기 위한 주요 툴 : Kustomize, Helm, argo CD
     + 배포 도구를 통해 Kubernetes에 배포

   - 운영 단계
     + 실제 운영 환경을 구성하는 요소와 툴
     + Nginx, Istio : 네트워크 트래픽 관리에 대한 도구
     + 인프라 환경 안에 설치가 되고 운영자가 이 툴들이 정상적으로 잘 동작하는지 확인하는 역할

   - 모니터링 단계
     + 주로 자원 사용량이나 App 로그, 트래픽 흐름을 확인
     + Grafana, Loki, Prometheus 등의 툴 사용
     + JAGER, ZIPKN : 트래픽 흐름을 보기 위한 도구
     + MSA 환경이 많아졌으므로 서비스들 간 트래픽이 어떻게 흘러가는지 추적하는게 중요해짐
     + 이러한 App들은 Kubernetes 클러스터 위에서 Pod 위에서 띄워져서 실행

3. Slack 사용 이유
   - DevOps의 중요 포인트를 차지하고 있는 툴들과 연동 가능
   - 파이프라인에서 발생하는 알람들을 알림 설정 가능
     
