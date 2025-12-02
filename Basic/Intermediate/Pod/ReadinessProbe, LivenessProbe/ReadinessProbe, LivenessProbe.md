-----
### ReadinessProbe, LivenessProbe
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/9d91ffab-6fa0-42c1-b994-b18e7cc5089d">
</div>

1. 사용 상황
   - Pod를 생성하면, 그 안에 컨테이너가 생기고 Pod와 Container가 생기고, Pod와 Container 상태가 Running이 되면서, 안에 있는 App도 정상적으로 구동 : Service에 연결이 되며, 이 서비스의 IP가 외부에 알려지면서 외부에서는 이 서비스를 통해 많은 사람들이 실시간으로 접근하게 되는 상황
   - 한 서비스에 두 개의 Pod가 연결되어 50%씩 트래픽을 나눠진다고 가정
     + Node2 서버가 다운이 되었을 때, Pod도 Failed가 되면서 사용자는 Pod1으로 100% 접속
     + Pod1이 이 트래픽을 감당할 수 있다면, App 서비스를 유지하는데 문제가 없음
     + Pod2의 경우 Auto-Healing 기능에 의해 다른 노드로 재생성되려고 할텐데, 그 과정에서 Pod와 Container가 Running 상태가 되면서 Service와 연결이 되는데, 아직 App이 구동 중인 순간이 발생
     + Service와 연결되자마자 트래픽이 Pod2로 유입되므로 App이 구동되고 있는 순간에는 사용자는 50% 확률로 에러 페이지를 보게 되는 문제 발생
   - 따라서, Pod를 만들 때 readinessProbe를 주게 되면, 해당 문제 해결 가능
     + App이 구동되기 전까지 Service와 연결이 되지 않게 해줌
     + 따라서, 사용 상황의 그림에서 Pod2의 상태는 Running이지만, 트래픽은 계속 Pod1에게만 전송되며, App이 완전히 준비된게 확인이 되면 Service와 연결이 되어 다시 트래픽은 50:50으로 흐르게 됨
   - App이 정상적으로 동작하고 있는데, 다시 App에 장애가 발생 (이 때 Pod의 상태는 Running)
     + 흔히 서비슬르 Tomcat으로 실행하면, Tomcat은 실행되고 있지만, 그 위에 App에 Memory Overflow 또는 문제가 생겨서 접속을 하면 500 Internal Server Error 발생
     + Tomcat 자체 프로세스가 죽은 것이 아닌 그 위를 돌고 있는 서비스에 문제가 발생한 것이므로 Tomcat 프로세스를 보고 있는 Pod 입장에서 계속 Running 상태에 있게 되고, 트래픽은 다시 문제가 됨
     + 이 때, App에 대한 장애 상황을 감지해주는 것이 Liveness Probe로, Pod를 만들 때 설정하게 되면 해당 App에 문제가 생기면 Pod를 재실행하게 만들어서 잠깐의 트래픽 에러가 발생하지만, 지속적 에러 상태가 발생하는 것을 방지
   - 즉, 서비스를 안정적으로 유지하기 위해 Readiness Probe를 사용해 App이 구동되는 순간에 트래픽 실패를 없애고, Liveness Probe를 사용해 App 장애 시 지속적 실패를 예방할 수 있음

<div align="center">
<img src="https://github.com/user-attachments/assets/64bd534f-6236-49da-8014-b494cb379ff1">
</div>

2. ReadinessProbe
   - 한 서비스가 Pod에 연결되어 있는 상태에서 Pod를 하나 더 생성 : Container에 HostPath로 Node에 Volume이 연결
   - 해당 컨테이너에 ReadinessProbe과 LivenessProbe를 설정하는데, 둘은 사용 목적이 틀릴 뿐, 공통적으로 들어가는 속성은 동일
     + httpGet, Exec, tcpSocket으로 해당 App에 대한 상태 확인 가능
       * httpGet의 경우 Port 번호나 Host 또는 Path, HTTP 헤더, Scheme를 체크할 수 있음
       * Exec는 특정 명령어를 전송해 그에 따른 결과를 확인
       * tcpSocket은 Port 번호와 Host을 확인해서 readinessProble나 livenessProbe에 대한 성공 여부를 확인해볼 수 있음
       * 셋 중 하나는 꼭 정해야 하는 속성
     + 옵션 값 존재
       * initialDelaySeconds : 최초 Probe를 하기 전 Delay 시간
       * periodSencods : Probe를 체크하는 시간 간격
       * timeoutSeconds : 지정된 시간까지 결과가 와야하는 시간
       * successThreshold : 몇 번 성공 결과를 받아야 성공으로 인정할 것인지 횟수 지정
       * failureThreshold : 몇 번 실패를 받아야 실패로 인정할 것인지 값 지정
       * 이 설정 값들은 별도로 넣지 않으면 default 값으로 지정

   - 예제에서는 Exec를 통해 Command ready.txt 파일을 조회하는 명령 사용
     + 옵션으로는 최초 딜레이 시간은 5초, 체크 간격은 10초, 세 번을 조회가 성공하면 정상적 구동된 것으로 간주
   - readinessProbe 설정을 하면, Pod를 만들 때, Node가 스케줄되고 이미지가 다운받아지면서 Pod와 Container 상태는 Running이 되지만, 이 Probe가 성공하기 전까지는 Condition의 containerReady 상태와 ready 상태는 false에서 변경되지 않음
     + 이 상태가 계속 false가 되면, End Point에서는 Pod의 IP를 NotReadyAddress로 간주하고, 서비스에 연결하지 않음
   - Kubernetes는 readinessProbe에 정의된 내용 대로 App 기동 상태 확인 : 컨테이너 상태가 Running이 되면 최초 5초 동안 지연을 하고 있다가, 5초가 지나면 ready.txt 파일이 있는지 확인 후, 없다면 10초 후 다시 확인하며, 계속 없다면 Pod의 Condition은 false로 유지
     + 만약, 이 노드에 ready.txt라는 데이터가 추가되면 Container와 Volume과 연결되어 readinessProbe를 확인할 때 성공 결과를 받음
     + 세 번 성공하게 되면, Condition의 상태는 true가 되고, EndPoint도 정상적으로 Address로 간주하면서 서비스와 연결

<div align="center">
<img src="https://github.com/user-attachments/assets/7a3f7d26-08ee-4b8e-b5e2-c97a1081ffa5">
</div>

3. LivenessProbe
   - 한 서비스에 두 Pod가 Running으로 구동되고 있는 상태
     + 이 중 한 App을 보면, /health라는 경로를 날리면 Status 200을 주면서 서비스가 정상적 운영 중이라는 health 체크가 형성
     + 이 컨테이너에는 livenessProbe가 설정되어 있고, 내용으로는 httpGet으로 path가 /health 경로를 확인
       + 옵션으로는 최초 5초 지연과 10초 간격 체크, 3번을 실패하게 되면 Pod가 재시작하도록 설정
   - Kubernetes가 HTTP GET으로 5초 후에 해당 path를 체크해보고, 200 OK를 받게 되고, 10초 후 또 체크를 해서 200 OK를 받으면 서비스가 정상 운영 중이라고 판단
   - 어느 순간 Path를 호출했는데, Internal Server Error를 발생하면서 500 Internal Server Error 발생 (서비스 장애) 하지만, Pod는 Running 상태
   - 3회를 Status 500을 받게 되면, Kubernetes는 문제가 있다고 판단해 이 Pod를 Restart
<div align="center">
<img src="https://github.com/user-attachments/assets/4f6a73ae-6747-4a60-8987-908e27b35bd5">
</div>
