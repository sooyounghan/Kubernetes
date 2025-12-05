-----
### Application 동작 중심의 Probe 이해
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/7a149156-acf6-4233-a6fc-e38dc49141c8" />
<img src="https://github.com/user-attachments/assets/0c0f34ee-dc6b-4c68-af5b-0ce6260db974" />
</div>

1. Kubernetes는 Application을 편하게 관리하기 위해 개발
2. Application 동작과 그 동작으로부터 자동화 되길 바라는 자동화 요구 사항이 존재
3. Application 동작
   - App 초기화 과정 : Pod가 생성되고, 컨테이너 내부 Jar 파일 실행되면서, 내부에 있는 스프링 초기화, DB 연결
     + DB 연결
     + Spring 초기화
     + Jar 실행

   - User 초기화
     + 초기 데이터 로딩
     + 연동 시스템 체크
     + DB 데이터 Validation

   - App 기동

4. 자동화 요구사항
   - App 초기화 과정
     + App을 초기화할 때는 시스템적으로 API를 받을 수 없으므로, App 상태 체크 기능 목적은 초기화가 다 끝났는지 학인 : Healthcheck API를 통해 200 OK 응답이 오는지 확인
     + 이 과정을 수행하는 동안 외부 API 금지시키는 기능 필요 

   - User 초기화
     + App이 기동되었으므로, API를 받을 수 있는 상태 : 필요한 App 상태 체크의 목적은 App이 살아있는지 확인 : Healthcheck API를 통해 실패되는지로 여부 확인
     + 외부 API에 대한 접근 금지

   - App 기동
     + 상태 체크 기능은 여전히 필요
     + 외부 API를 자동으로 허용시켜주는 기능 필요

   - App 장애
     + App 상태 체크 기능에 의해 App을 재기동시켜주는 기능이 필요

5. Kubernetes 제공 기능
   - Probe 라는 속성이 자동화 요구 사항의 부합하는 기능 제공
   - App 초기화
     + Kubernetes가 API를 계속 전송하면서 App 초기화 판단
     + Service와 Pod는 selector와 label로 설정했지만, Kubernetes는 아직 연결해놓지 않음
     + startupProbe가 성공하면 해당 Probe를 중단시키고, livenessProbe와 readnessProbe 기능을 활성화

   - User 초기화
     + livenessProbe는 설정된 API를 계속 호출하면서 App 기동 여부 판단, API가 실패하면 장애라 판단하고 재기동
     + readnessProbe는 App에 API 결과가 실패가 되는 경우 Service를 Pod에 연결하지 않고, 성공하게 되는 순간 Kubernetes가 두 Object를 연결하여 트래픽이 전송
       * 중간에 실패하는 상황이 발생하면, Kubernetes는 연결 해제 후, 다시 성공하면 연결
       * 💡 User 초기화 과정에서 App은 초기화되었지만, 의도적으로 외부 트래픽을 받지 않게 할 때 사용
       * 결과 : 의도했던 초기화 작업들이 모두 완료됨을 알려주는 것
     + 단, 두 Probe는 App이 종료될 때까지 계속 HealthCheck해야 하므로, 가볍게 로직을 구성해야 함

-----
### API를 전송하며 Probe 동작 확인
-----
<img width="1581" height="571" alt="image" src="https://github.com/user-attachments/assets/7c98e5c4-06e0-4919-9ed4-8ca10211092e" />

: Master Node에서 실행
```bash
﻿// 1번 API - 외부 API 실패
curl http://192.168.56.30:31231/hello
```

```bash
﻿// 2번 API 
// 외부 API 실패
curl http://192.168.56.30:31231/hello

// 내부 API 성공
kubectl exec -n anotherclass-123 -it api-tester-1231-7459cd7df-2hdhk -- curl localhost:8080/hello
kubectl exec -n anotherclass-123 -it <my-pod-name> -- curl localhost:8080/hello
```

```bash
﻿// 3번 API - 외부 API 성공
curl http://192.168.56.30:31231/hello
```

```bash
﻿// 4번 API
// 트래픽 중단 - (App 내부 isAppReady를 False로 바꿈)
curl http://192.168.56.30:31231/traffic-off

// 외부 API 실패
curl http://192.168.56.30:31231/hello

// 트래픽 재개 - (App 내부 isAppReady를 True로 바꿈)
kubectl exec -n anotherclass-123 -it api-tester-1231-7459cd7df-2hdhk -- curl localhost:8080/traffic-on
```

```bash
﻿// 5번 API - 장애발생 (App 내부 isAppLive를 False로 바꿈)
curl http://192.168.56.30:31231/server-error
```
