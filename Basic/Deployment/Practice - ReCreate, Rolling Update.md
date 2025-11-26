-----
### 실습 - ReCreate, Rolling Update
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/8b327a75-a6ee-47d3-93d5-301cd99ece56">
</div>

1. ReCreate
   - Update 시 Downtime 발생 (서비스 일시 중지)
   - Update 시 추가적인 자원 요구되지 않음
   - template 내용 수정시 자동으로 업그레이드 됨
   - Update 시 기존 ReplicaSet의 Replica를 0으로 만들고, 새 ReplicaSet을 만들면서 Replica를 2로 해줌
   - 계속 업그레이드시 ReplicaSet는 누적되고, revisionHistoryLimit(Default:10)으로 개수 관리 가능
   - 💡 Deployment는 ReplicaSet별로 Pod와의 추가적인 Selector와 Label(pod-template-hash)를 만들어 주므로, ReplicaSet가 타 Pod를 연결할 가능성이 없음
   - Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-1
spec:
  selector:
    matchLabels:
      type: app
  replicas: 2
  strategy:
    type: Recreate
  revisionHistoryLimit: 1
  template:
    metadata:
      labels:
        type: app
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
      terminationGracePeriodSeconds: 10
```
   - Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-1
spec:
  selector:
    type: app
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
```
   - Master Node에서 1초 단위로 Service IP 호출
```bash
while true; do curl <service-ip>:8080/version; sleep 1; done
```
```bash
[root@k8s-master ~]# while true; do curl 10.101.58.211:8080/version; sleep 1; done
Version : v1
Version : v1
Version : v1
Version : v1
Version : v1
Version : v1
Version : v1
...
```

   - app:v2로 업그레이드
     + Dashboard : Deployment > Edit > template의 spec 수정
   - kubectl
```bash
// kubectl set image deployment <deployment-name> <container-name>=<image>
kubectl set image deployment deployment-1 container=kubetm/app:v2
```
<div align="center">
<img src="https://github.com/user-attachments/assets/a1272738-ce5d-45a4-b5e7-01c5ff055160">
</div>

   - 이 상태에서 V3으로 업데이트하면, V1의 ReplicaSet은 삭제

   - Kubectl 명령으로 Rollback
```bash
kubectl rollout history deployment deployment-1
```
```bash
[root@k8s-master ~]# kubectl rollout history deployment deployment-1
deployment.apps/deployment-1 
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
```

```bash
kubectl rollout undo deployment deployment-1 --to-revision=2
```
```bash
[root@k8s-master ~]# kubectl rollout undo deployment deployment-1 --to-revision=2
deployment.apps/deployment-1 rolled back
```

2. RollingUpdate
    - Update시 Downtime 없음 (서비스 중단 없음)
    - Update시 추가적인 자원이 요구됨
    - 기존 ReplicaSet의 Replica를 하나씩 줄이고, 새 ReplicaSet의 Replica를 하나씩 늘리면서 Update가 진행됨
    - Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-2
spec:
  selector:
    matchLabels:
      type: app2
  replicas: 2
  strategy:
    type: RollingUpdate
  minReadySeconds: 10
  template:
    metadata:
      labels:
        type: app2
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
      terminationGracePeriodSeconds: 0
```
   - Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-2
spec:
  selector:
    type: app2
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
```

  - Master Node에서 1초 단위로 Service IP 호출
```bash
while true; do curl <service-ip>:8080/version; sleep 1; done
```
```bash
[root@k8s-master ~]# while true; do curl 10.97.53.150:8080/version; sleep 1; done
Version : v1
Version : v1
...
Version : v1
Version : v1
Version : v2
Version : v2
Version : v1
Version : v2
Version : v2
Version : v2
Version : v2
Version : v2
...
Version : v2
Version : v2
Version : v2
Version : v2
...
```

3. Blue/Green
   - Update 시 Downtime 없음 (서비스 중단 없음)  
   - Update 시 추가적인 자원이 요구됨
   - ReplicaSet
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica1
spec:
  replicas: 2
  selector:
    matchLabels:
      ver: v1
  template:
    metadata:
      labels:
        ver: v1
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
      terminationGracePeriodSeconds: 0
```
   - Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-3
spec:
  selector:
    ver: v1
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
```
  - Master Node에서 1초 단위로 Service IP 호출
```bash
while true; do curl <service-ip>:8080/version; sleep 1; done
```
  - Update할 새 ReplicaSet(replica2) 생성 후 Service(svc-3)의 Selector를 변경하여 트래픽 변경

   - ReplicaSet
```bash
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica2
spec:
  replicas: 2
  selector:
    matchLabels:
      ver: v2
  template:
    metadata:
      labels:
        ver: v2
    spec:
      containers:
      - name: container
        image: kubetm/app:v2
      terminationGracePeriodSeconds: 0
```
   - Service의 Selector 수정 (v1->v2)
```bash
apiVersion: v1
kind: Service
metadata:
  name: svc-3
spec:
  selector:
    ver: v2
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
```
```bash
[root@k8s-master ~]# while true; do curl 10.106.233.178:8080/version; sleep 1; done
Version : v1
Version : v1
Version : v1
Version : v1
....
Version : v1
Version : v1
Version : v1
Version : v1
Version : v1
Version : v1
Version : v2
Version : v2
Version : v2
Version : v2
...
```
   - 실습 후 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 or Master Node에서 아래 명령 실행)
```bash
kubectl delete deploy deployment-1 deployment-2
kubectl delete rs replica1 replica2
kubectl delete svc svc-1 svc-2 svc-3
```
