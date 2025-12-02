-----
### DaemonSet, Job, CronJob
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/491d526c-a013-4786-aba2-1070191f8270">
</div>

1. DaemonSet
   - 각 노드에 자원이 다르게 남아있는 상태에서 자원 상태와 상관없이 모든 노드에 Pod가 하나씩 생긴다는 특징이 있음
   - 만약 노드가 10개라면, 각 노드에 하나씩 총 10개의 Pod가 생김
   - 성능 수집 서비스 : 각 노드들의 성능 상태를 모두 감시하는 것
     + 모니터링 대시보드가 있으며, 각 노드에 프로메테우스와 같은 성능 수집 에이전트가 설치되어야 모든 노드들의 정보들을 모니터링 시스템에 전달 가능
   - 로그 수집 : 특정 노드에 장애가 발생했을 경우 문제를 파악하려면 로그를 봐야함
     + Fluentd와 같은 서비스는 각 노드에 설치되어 로그 정보들을 수집
   - 노드들을 스토리지에 활용할 수 있도록 ClusterFS와 같은 각 노드에 설치되서 해당 자원을 가지고 네트워크 파일 시스템 구축 가능
   - Kubernetes 자체도 네트워킹 관리를 하기 위해 각 노드에 DaemonsSet으로 Proxy 역할을 하는 Pod를 생성

2. Job
   - Controller에 의해 만들어진 Pod들은 Controller에 의해 장애가 감지되면, 다른 노드에 재생성되므로 서비스가 계속 유지
   - Replicas에서 만들어진 Pod들은 일을 하지 않으면 Pod를 다시 Restart하므로 이 Pod에 있는 서비스는 서비스가 유지되는 목적으로 사용
   - ReCreate와 Restart
     + ReCreat : Pod를 다시 만들어주므로 Pod의 이름이나 IP들이 변경
     + Restart : Pod는 그대로 있고, Pod 안의 컨테이너만 재가동
   - Job의 경우에는 프로세서가 일을 하지 않으면 Pod가 종료 (종료 : Pod가 삭제되는 것이 아닌 자원을 사용하지 않는 상태로 멈춰있음)
     + Pod 안의 로그를 확인해, 필요가 없으면 직접 삭제

3. CronJob
   - 주기적인 시간에 따라 Job을 생성하는 역할
   - 대체로 Job을 하나의 단위로 쓰지 않고, CronJob을 만들어 특정 시간에 반복적으로 실행할 목적으로 사용
   - 예) 새벽마다 정기적 데이터(DB) 백업 및 주기적인 업데이트, 예약 메일이나 SMS 같은 메세지 발송

-----
### DaemonSet, Job, CronJob 특징
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/7c940952-c14d-4d85-8d3e-50603a1701fe">
</div>

1. DaemonSet
   - selector와 template이 있어 모든 노드에 template으로 Pod를 만들고, selector와 label로 DaemonSet과 연결
   - 만약 노드들의 OS 종류가 달라서, Label이 그림과 같이 설정되었다고 가정
     + centosOS의 Pod는 Ubuntu에서 실행할 수 없으므로, 이 노드에는 설치를 안하고 싶다면, 이 파드에 NodeSelector라고 지정해주면, 해당 Label이 없는 노드에는 Pod가 생성되지 않음
     + 즉, DaemonSet은 한 노드에 하나를 추가해서 Pod를 만들 수 없지만, 노드에 Pod를 생성 방지는 가능
   - 그리고 특정 노드로 접근했을 때, 이 노드에 들어있는 Pod에 접근이 되도록 많이 사용 : HostPort로 설정하여 직접 노드가 있는 Port가 해당 Pod로 연결
   - Yaml 파일
     + selector, template이 존재
     + template 안에는 nodeSelector를 지정하면 해당 label이 붙은 노드에만 Pod가 생기고, 지정하지 않으면 모든 노드에 Pod가 생성
     + containterPort가 존재하고, hostPort를 지정 가능 : 18080번 Port로 들어온 트래픽은 해당 Pod에 8080번이라는 Container Port로 연결

2. Job
   - template과 selector가 존재하는데, 이 template은 특정 작업만 하고 종료가 되는 일을 Pod들을 저장 (해당 selector는 직접 만들지 않아도 Job이 자동 생성)
   - template을 가지고 하나의 Pod를 일반적으로 생성하고, 그 Pod가 일을 다하면, Job도 종료가 됨
   - Yaml 파일
     + completions: 6 - 6개의 Pod를 순차적으로 실행시켜서 모두 작업이 끝나야 Job도 종료
     + parallelism: 2 - Pod가 2개씩 생성
     + activeDeadLineSeconds: 30 - Job은 30초 후 기능 정지되며, 실행되고 있는 모든 Pod들은 삭제되고, 실행되지 못한 Pod들도 실행 불가
       * 예) 10초 걸릴 일에 Job을 만들었는데 30초가 되도록 Job이 끝나지 않으면 Hang에 걸릴 확률이 크므로, 이럴 경우 Pod들을 삭제해 자원을 Release하고 더 이상 작업을 진행하지 않도록 설정할 때 사용
     + restartPolicy: Never (필수값) - Job의 경우 Never와 OnFailure만 사용 가능

3. CronJob
   - Job Template이 있어서 해당 내용을 통해 Job을 생성해주고, Schedule이 있어 이 시간을 주기로 Job을 만듬 :
   - Cron 포맷 (schedule: "*/1 * * * *") - 1분에 하나의 Job을 생성한다는 의미로, 1분 간격으로 Job이 생성되고 Job은 또 자신의 역할인 Pod를 생성
   - concurrencyPolicy라는 기능 존재 : Allow, Forbid, Replace
     + Allow는 1분 간격으로 스케줄을 한다고 설정하면, 1분이 됐을 때 Job을 생성하고 Pod가 생성 (Default)되며, 2분이 되면 사전에 만들어진 Pod가 어떠한 상태 간에 상관없이 자신의 Schedule Time이 되면 새로운 Job을 만들고, Pod 생성
     + Forbid는 1분이 됐을 때 Job이 생성되지만, 2분이 됐을 때까지 Pod가 종료되지 않고 계속 실행이 되고 있으면 2분쨰 생겨야 되는 Job은 Skip이 되며, Pod가 종료되는 즉시 다음 스케줄 타임에 있는 Job 생성
     + Replace는 1분의 Job이 만들어졌고, 2분 Schedule이 되었는데, 옵션을 통해 CronJob을 정교하게 다룰 수 있음
<div align="center">
<img src="https://github.com/user-attachments/assets/24e5fd9d-6cb4-472b-b5b8-c3694f3a4c4a">
</div>

