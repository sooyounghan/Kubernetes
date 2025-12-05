-----
### 일시적 장애 상황에서의 Probe 활용
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/4078b766-15fb-417f-b93a-1a901c5442cc" />
</div>

1. App이 잘 기동되어 서비스 중인 상황에서, 일시적 장애 발생
   - Overload (App 부하 증가)
   - Leak
   - Thread Full

2. livenessProbe와 readnessProbe가 실패하게 되면, Kubernetes는 Pod를 재기동
   - 오히려 이 기능이 없었다면 정상으로 돌아올 수 있는 가능성도 존재


3. 해소 방법
   - 일시적 부하 상황에서 readlinesspProbe는 실패 시 외부 API 금지시키고 App에 추가적 부하를 감소시키므로 그대로 둬도 무방
   - livenessProbe의 경우, readlinessProbe와의 주기를 같지 않게 하여, 실패하는 데 걸리는 시간을 길게 설정해 Pod가 쉽게 재가동 되는 것을 방지
   - 중요한 시스템들은 보통 Pod을 이중화시켜놓으므로, 일시적 장애가 있는 Pod들을 빨리 재기동 시키는 것이 꼭 정답이 아님 : 부하가 있는 Pod에 시간을 줘서 스스로 해결하도록 설정하는 것이 좋음
   
