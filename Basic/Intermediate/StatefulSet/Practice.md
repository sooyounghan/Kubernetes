-----
### StatefulSet - Pod, PersistentVolume, Headless Service
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/3744aa3c-054b-4bb4-bf0b-8fca0fed1cd3">
</div>

1. StatefulSet Controller 
   ​ - Stateless App은 Pod들이 모두 똑같은 역할을 하는 데 반해, Stateful App은 각각의 Pod가 자신의 역할이 존재
    - Pod 이름 : ReplicaSet (ReplicaSet에 랜덤문자 포함) , StatefulSet (StatefulSet에 0부터 순차적으로 숫자 부여)
    - Pod 생성 (replica:3) : ReplicaSet (Pod 3개가 동시 생성) , StatefulSet (0부터 순차적으로 하나씩 생성)
    - Pod 삭제 (replica:3) : ReplicaSet (Pod 3개가 동시 삭제) , StatefulSet (2부터 순차적으로 하나씩 삭제) 
    - ReplicaSet
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica-web
spec:
  replicas: 1
  selector:
    matchLabels:
      type: web
  template:
    metadata:
      labels:
        type: web
    spec:
      containers:
      - name: container
        image: kubetm/app
      terminationGracePeriodSeconds: 10
```
   - StatefulSet 
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: stateful-db
spec:
  replicas: 1
  selector:
    matchLabels:
      type: db
  template:
    metadata:
      labels:
        type: db
    spec:
      containers:
      - name: container
        image: kubetm/app
      terminationGracePeriodSeconds: 10
```

2. PersistentVolumeClaim
    - Volume 생성 (replica:3) : ReplicaSet (수동으로 하나의 공유 Volume 생성) , StatefulSet (Pod별 각각의 Volume이 동적으로 생성) 
    ​- Volume 연결 (replica:3) : ReplicaSet (3Pod가 한 Volume에 연결) , StatefulSet (Pod별 각각의 Volume 생성)
    - StatefulSet은 Pod 삭제시 Volume은 그대로 있고, 새 Pod가 해당 Volume에 연결되어 같은 역할 수행
    - PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: replica-pvc1
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1G
  storageClassName: "fast"
```
   - ReplicaSet
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica-pvc
spec:
  replicas: 3
  selector:
    matchLabels:
      type: web2
  template:
    metadata:
      labels:
        type: web2
    spec:
      nodeSelector:
        kubernetes.io/hostname: k8s-worker1
      containers:
      - name: container
        image: kubetm/init
        volumeMounts:
        - name: longhorn
          mountPath: /applog
      volumes:
      - name: longhorn
        persistentVolumeClaim:
          claimName: replica-pvc1
      terminationGracePeriodSeconds: 10
```
   - 특정 Pod에 들어가서 파일 생성
```bash
touch /applog/server.log
```
```bash
[root@replica-pvc-ncv5b /]# touch /applog/server.log
[root@replica-pvc-ncv5b /]# cd /applog
[root@replica-pvc-ncv5b applog]# ls
lost+found  server.log
```

   - 다른 Pod에 들어가서 파일 확인 (공유 Volume이기 때문에 데이터 공유됨)
```bash
ls /applog/server.log
```
```bash
[root@replica-pvc-hpcrx /]# ls /applog/server.log
/applog/server.log
```

   -  StatefulSet
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: stateful-pvc
spec:
  replicas: 2
  selector:
    matchLabels:
      type: db2
  serviceName: "stateful-headless"
  template: 
    metadata:
      labels:
        type: db2
    spec:
      containers:
      - name: container
        image: kubetm/app
        volumeMounts:
        - name: volume # Volume Mount 한 Name과 동일해야 함
          mountPath: /applog
      terminationGracePeriodSeconds: 10
  volumeClaimTemplates:
  - metadata:
      name: volume # 이 부분과 동일
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1G
      storageClassName: "fast"
```
   - 특정 Pod에 들어가서 파일 생성
```bash
touch /applog/server.log
```
```bash
root@stateful-pvc-0:/# cd /applog
root@stateful-pvc-0:/applog# touch server1.log
root@stateful-pvc-0:/applog# ls
lost+found  server1.log
root@stateful-pvc-0:/appl
```

   - 다른 Pod에 들어가서 파일 확인 (각자의 Volume이기 때문에 데이터 공유 안됨)
```bash
ls /applog/server.log
```
```bash
root@stateful-pvc-1:/# cd /applog/
root@stateful-pvc-1:/applog# ls
lost+found
```

3. Headless Service
    - Stateful App은 통상 DB 서버의 역할을 함   
    - StatefulSet은 Headless Service를 통해 ```도메인 이름<pod-name.server-name>```으로 각각의 Pod에 연결 가능
    - Service (Headless) 
```yaml
apiVersion: v1
kind: Service
metadata:
  name: stateful-headless
spec:
  selector:
    type: db2
  ports:
    - port: 80
      targetPort: 8080    
  clusterIP: None
```
  - Pod 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: request-pod
spec:
  containers:
  - name: container
    image: kubetm/init
```

   - Pod(request-pod)에 들어가서 StatefulSet의 Pod 호출
```bash
nslookup stateful-headless
curl stateful-pvc-0.stateful-headless:8080/hostname
curl stateful-pvc-1.stateful-headless:8080/hostname
```
   - 실습 후 모든 리소스 삭제 ​(Dashboard에서 리소스별 삭제 or Master Node에서 아래 명령 실행)
```bash
kubectl delete pod request-pod
kubectl delete svc stateful-headless
kubectl delete rs replica-web replica-pvc
kubectl delete sts stateful-db stateful-pvc
kubectl delete pvc replica-pvc1 volume-stateful-pvc-0 volume-stateful-pvc-1 volume-stateful-pvc-2
```
