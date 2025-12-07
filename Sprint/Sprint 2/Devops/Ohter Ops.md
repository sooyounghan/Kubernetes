-----
### DevOps 외 다른 Ops들
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/9455edec-605d-40e2-9ede-8c7f16847984" />
</div>

1. GitOps : DevOps 파이프라인을 Git 하나로 통일
   - Git은 이슈나 협업 관리, 빌드 / 테스트 / 배포까지 모두 제공되는 기능 제공
   - Github에는 Github Actions 존재 (Jenkins 대체 가능, 테스트나 빌드 툴 모두 돌릴 수 있음)

2. DevSecOps : 빠른 배포와 보안을 동시에 잡는 것
   - DevOps의 보안적 요소까지 자동화
   - 예) 자동화 Jenkins 빌드 전, Git에서 다운받은 소스코드에 심각한 버그가 없는지 검사 (SonarQube를 일반적으로 사용)
   - 또, 이미지를 만들 때, 이미지에 jar 파일 뿐만 아니라, 라이브러리를 추가하는데, 취약한 라이브러리에 대해 DockerHub에 컨테이너 이미지를 올릴 때, 취약점 검사를 자동으로 실시하도록 설정 가능
   - Kubernetes Cluster에 대한 보안 툴들도 다양하게 존재
   - DevOps는 파이프라인 중간에 보안 검사를 자동화시켜 놓는 것

3. MLOps : 머신러닝, AI 분야를 위한 DevOps
   - 데이터 분석과 개발자 간 커뮤니케이션을 잘 하도록 파이프라인을 생성

4. LLMOps : MLOps와 비슷하게 AI 분야로, 방대한 규모의 머신러닝을 하기 위한 특화된 DevOps
5. FinOps : Fin(Finance)로, 클라우드 환경이 좋지만 비용이 생각보다 많이 발생하므로, 클라우드 환경을 사용할 때 비용을 절감하는데 초점을 두고 파이프라인을 만든 것

-----
### DevOps 한방 정리
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/afc34711-9f62-4b6c-9941-3f67e6558ee7" />
</div>
