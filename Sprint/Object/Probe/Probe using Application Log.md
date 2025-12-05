-----
### Application 로그를 통한 Probe 동작 분석
-----
1. Applicaiton 로그를 통한 프로브 동작 분석
  - 실습 전 사전 준비작업
```bash
// HPA minReplica 1로 바꾸기 - Master Node
kubectl patch -n anotherclass-123 hpa api-tester-1231-default -p '{"spec":{"minReplicas":1}}'
```
   - Grafana 접속해서 Loki에 Pod로그 화면 세팅
```bash
// Grafana 접속 IP
﻿http://192.168.56.30:30001/
```
<div align="center">
<img src="https://github.com/user-attachments/assets/5daea464-d531-489e-94c1-2973291939ba" />
</div>

  - Pod 삭제
<div align="center">
<img src="https://github.com/user-attachments/assets/141c57c5-d62e-4322-9168-b8881c1d4a63" />
</div>

2. Applicaiton 로그 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/759bc1d1-425c-4ceb-8697-ed3540f3f6c3" />
</div>

  - [App 초기화] 중 시스템 연동 작업을 하고, [User 초기화] 중에 초기 데이터 로딩 작업이 있으며, 이 작업이 다 끝나야 [기동완료] 가 되는 App 기동 상황
  - startupProbe가 5초 간격으로 호출 되고, 성공할 때까지 반복, 성공 후 readinessProbe와 livenessProbe가 시작
  - livenessProbe는 바로 성공을 하지만, readinessProbe는 처음 실패 후에 [기동완료] 가 되서야 성공을 함

- livenessProbe와 readinessProbe는 10초 간격으로 계속 호출 됨
