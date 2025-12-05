-----
### Application Log를 통한 Probe 동작 분석
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/a3b4c90e-a24a-49b1-adda-adc9dedeff28" />
</div>

1. Pod의 Probe 설정
   - startupProbe, readiessProbe, livenessProbe 3가지 존재 : /ready URL을 8080포트에 10초 간격 전송
   - 각 성공(suceesThreshold), 실패(failureThreshold)일 때 수치 : 1, 10
   - Pod가 생성되자마자 Probe 기능 동작
     + 처음 App 가동 중 : Kubernetes가 startupProbe을 동작하면서 Object 속성에 있는대로 10초마다 /ready API 전송
       * 기동 중일 때는 응답을 받을 수 없으며, 계속 실패 되며, 10번을 실패하기 전 한 번이라도 응답이 있으면 성공으로 간주
       * 이후, startupProbe 기능 중지, readinessProbe, livenessProbe 기능 동작
     + 두 Probe는 /ready API를 10초 간격으로 전송 : App이 실행되는 동안 200 OK라는 결과를 반환하여 Probe 동작 반복
       + readinessProbe : 성공했을 때 외부 트래픽을 Pod가 받을 수 있는 상태로 만들어주면서 서비스 활성화
       + livenessProbe : App이 실행되고 있는지 확인하면서, 장애가 발생하면 API는 실패하게 되고, 설정에 따라 두 번 실패하면 Kubernetes 앱 재기동

2. 일반적으로는 App 기동 시간에 따라 startupProbe 실패 횟수만 조정해서 사용
