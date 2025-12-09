-----
### 기초부터 Blue / Green 까지 배포 구축 단계
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/257e7a2c-15d3-48ba-98e3-e58b7a07b26a" />
</div>

1. Step 1 : Jenkins Pipeline 기본 구성 만들기
   - GitHub에서 릴리즈 파일들을 가져와서 빌드 / 배포하는 구성을 Jenkins 파이프라인으로 생성

2. Step 2 : GitHub 연결과 파이프라인 세분화
   - JenKins 파이프라인을 쓰는 이유 : 파이프라인 스크립트를 GitHub에서 가져오고, Pipeline을 세분화
   - 세분화함으로써 각 구간별로 시간이 얼마나 걸렸는지 바로 보임
  
3. Step 3 : Blue / Green 배포
   - Blue가 배포된 상태에서 Greent 배포
   - V2 버전의 Deployment가 생성이 되면 트래픽을 V2로 전환
   - Service에서 select를 변경 후, V1 Deployment를 사제하면 Blue 배포가 삭제
   - 단, 운영에서만 테스트 가능한 경우를 한 번 해볼 것이며, 다음으로 수동 배포 시 Roll-Back이 빠르다는 것 확인

4. Step 4 : 버튼 한 번으로 자동 배포가 되는 스크립트 생성
