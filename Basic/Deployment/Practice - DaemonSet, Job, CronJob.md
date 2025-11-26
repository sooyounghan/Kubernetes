-----
### Practice - DaemonSet, Job, CronJob
-----

<img width="886" height="232" alt="image" src="https://github.com/user-attachments/assets/95f8604b-f174-49f3-a1c4-0e6555e7e123" />

1. DaemonSet
   - 모든 Node에 Pod를 하나씩 생성시킴 (예) 노드별 성능 / 로그 수집, Volume 솔루션)
   - Pod의 hostPort를 설정하면 각 Node별 hostPort가 해당 노드위에 있는 Pod의 Port로 연결됨
   - Pod의 NodeSelector를 통해 원하는 Node에만 Pod를 배치시킬 수 있음
   - DaemonSet (hostPort)
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-1
spec:
  selector:
    matchLabels:
      type: app
  template:
    metadata:
      labels:
        type: app
    spec:
      containers:
      - name: container
        image: kubetm/app
        ports:
        - containerPort: 8080
          hostPort: 18080 # 해당 18080 포트로 접근하면 8080 포트인 Pod로 트래픽이 전송
```
   - Master Node에서 명령어 호출
```bash
curl 192.168.56.31:18080/hostname
curl 192.168.56.32:18080/hostname
```
```bash
[root@k8s-master ~]# curl 192.168.56.31:18080/hostname
Hostname : daemonset-1-2hfcx
[root@k8s-master ~]# curl 192.168.56.32:18080/hostname
Hostname : daemonset-1-7dnfv
```

   - DaemonSet (NodeSelector)
     +  Worker Node에 라벨 생성
```bash
kubectl label nodes k8s-worker1 os=centos
kubectl label nodes k8s-worker2 os=ubuntu
```
   - DaemonSet
```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-2
spec:
  selector:
    matchLabels:
      type: app
  template:
    metadata:
      labels:
        type: app
    spec:
      nodeSelector:
        os: centos
      containers:
      - name: container
        image: kubetm/app
        ports:
        - containerPort: 8080
```

2. Job
   - 일시적인 작업을 하기 위한 용도로 Pod를 생성하고, 작업이 종료되면 Pod는 중지됨 (바로 삭제되지 않음)
   - Job1
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-1
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: container
        image: kubetm/init
        command: ["bash", "-c", "echo 'job start';sleep 20; echo 'job end'"] # start라는 메세지를 찍고 20초 동안 Sleep을 줘서 Pod가 작업중이도록 작동
      terminationGracePeriodSeconds: 0
```
   - Job2
     + completions : 총 생성 시킬 Pod 수
     + parallelism : 동시에 생성 시킬 Pod 수
     + activeDeadlineSeconds : 30초가 될 때까지 모든 작업이 완료되지 않으면 실행중인 Pod는 강제로 삭제되, 완료된 Pod만 남아있게 됩니다.
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-2
spec:
  completions: 6
  parallelism: 2
  activeDeadlineSeconds: 40
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: container
        image: kubetm/init
        command: ["sh", "-c", "echo 'job start';sleep 20; echo 'job end'"]
      terminationGracePeriodSeconds: 0
```

3. CronJob
  - 특정 작업시간마다 반복적인 작업을 하기 위해 Job을 생성시킴 (예) 새벽시간마다 DB 백업, 주기적인 업데이트 체크, 예약 메일 / SMS 전송) 
  - schedule : (*/1 * * * *) 1분 간격으로 Job을 하나씩 생성  
  - suspend : 일시 정지
  - Manual Trigger : 수동으로 Job을 직접 실행 시킴 (기존에는 manual로 만든 Job이 삭제가 안됐지만, 현재는 삭제되도록 변경)
  - CronJob
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cron-job
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: container
            image: kubetm/init
            command: ["bash", "-c", "echo 'job start';sleep 20; echo 'job end'"]
          terminationGracePeriodSeconds: 0
```
   - Manual
```bash
kubectl create job --from=cronjob/cron-job cron-job-manual-001
```

   - Suspend (true : 일시정지)
```bash
kubectl patch cronjobs cron-job -p '{"spec" : {"suspend" : true }}'
```

   - CronJob (ConcurrencyPolicy)
     + allow (default) : 이전 Job의 상태에 상관없이 정해진 시간에 따라 Job을 생성
     + forbid : 동시에 Job이 실행되는걸 허용하지 않음. 이전 Job이 아직 실행중이면 정해진 시간의 Job은 Skip됨
     + replace : 다음 정해진 시간까지 이전 Job이 실행중이면, 강제로 종료시키고 새로운 Job을 생성시킴
   - forbid
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cron-job-2
spec:
  schedule: "20,21,22 * * * *" # 20 21 22 분에 잡 생성
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: container
            image: kubetm/init
            command: ["bash", "-c", "echo 'job start';sleep 140; echo 'job end'"] # 2분 20초 동안 Running 중인 Pod 생
          terminationGracePeriodSeconds: 0
```
   - replace
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cron-job-3
spec:
  schedule: "20,21,22 * * * *"
  concurrencyPolicy: Replace
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: container
            image: kubetm/init
            command: ["bash", "-c", "echo 'job start';sleep 140; echo 'job end'"]
          terminationGracePeriodSeconds: 0
```
   - 실습 후 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 or Master Node에서 아래 명령 실행)​
```bash
kubectl delete ds daemonset-1 daemonset-2
kubectl delete job job-1 job-2
kubectl delete cronjob cron-job cron-job-2 cron-job-3

// Node에 라벨 삭제
kubectl label nodes k8s-worker1 os-
kubectl label nodes k8s-worker2 os-
```
