-----
### Readinessprobe, Livenessprobe
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/bee3c699-fa32-4117-ab53-579ef898aa7a">
</div>

1. ReadinessProbe
   - Pod가 생성되어도 App이 구동되는 시간동안 트래픽을 받지 않도록 해줌
   - initialDelaySeconds : Pod 생성후 해당 지연시간이 지난 후에 헬스체크 시작
   - periodSeconds : 10초 간격으로 헬스 체크
   - successThreshold : 성공시 Service와 연결됨
   - failureThreshold : 실패시 재시도
   - Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-readiness
spec:
  selector:
    app: readiness
  ports:
  - port: 8080
    targetPort: 8080
```
   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    app: readiness  
spec:
  containers:
  - name: container
    image: kubetm/app
    ports:
    - containerPort: 8080	
  terminationGracePeriodSeconds: 0
```
   - Master Node에서 Service IP로 호출 (1초 단위)​​
```bash
while true; do date && curl <service-ip>:8080/hostname; sleep 1; done
```

  - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-readiness-exec1
  labels:
    app: readiness  
spec:
  containers:
  - name: readiness
    image: kubetm/app
    ports:
    - containerPort: 8080	
    readinessProbe: # readinessProbe
      exec: # exec 커맨드 : ready.txt 파일 읽음
        command: ["cat", "/readiness/ready.txt"]
      initialDelaySeconds: 5
      periodSeconds: 10
      successThreshold: 3
    volumeMounts:
    - name: host-path
      mountPath: /readiness
  volumes:
  - name : host-path
    hostPath: # Volume과 Node을 HostPath로 연결
      path: /tmp/readiness 
      type: DirectoryOrCreate
  terminationGracePeriodSeconds: 0
```
   - 새 Master Node 탭을 열어서 Event 로그 확인
```bash
// 모든 Event 정보를 지속적으로 조회해서 | 그중에 pod-readiness-exec1라는 단어와 매칭되는 내용만 출력
kubectl get events -w | grep pod-readiness-exec1
```
```bash
[root@k8s-master ~]# kubectl get events -w | grep pod-readiness-exec1
2m37s       Normal    Pulling                   pod/pod-readiness-exec1   Pulling image "kubetm/app"
2m35s       Normal    Pulled                    pod/pod-readiness-exec1   Successfully pulled image "kubetm/app" in 2.218519645s (2.218604884s including waiting)
2m35s       Normal    Created                   pod/pod-readiness-exec1   Created container readiness
2m35s       Normal    Started                   pod/pod-readiness-exec1   Started container readiness
2m34s       Normal    Killing                   pod/pod-readiness-exec1   Stopping container readiness
12s         Normal    Scheduled                 pod/pod-readiness-exec1   Successfully assigned default/pod-readiness-exec1 to k8s-worker2
10s         Normal    Pulling                   pod/pod-readiness-exec1   Pulling image "kubetm/app"
8s          Normal    Pulled                    pod/pod-readiness-exec1   Successfully pulled image "kubetm/app" in 2.313895923s (2.313906492s including waiting)
8s          Normal    Created                   pod/pod-readiness-exec1   Created container readiness
8s          Normal    Started                   pod/pod-readiness-exec1   Started container readiness
1s          Warning   Unhealthy                 pod/pod-readiness-exec1   Readiness probe failed: cat: /readiness/ready.txt: No such file or directory
...
```

   - 새 Master Node 탭을 열어서 Pod와 Service 상태 확인
```bash
kubectl describe pod pod-readiness-exec1 | grep -A5 Conditions
```
```bash
[root@k8s-master ~]# kubectl describe pod pod-readiness-exec1 | grep -A5 Conditions
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
```
   - Ready : False 상태
```bash
kubectl describe endpoints svc-readiness
```
```bash
[root@k8s-master ~]# kubectl describe pod pod-readiness-exec1 | grep -A5 Conditions
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
[root@k8s-master ~]# kubectl describe endpoints svc-readiness
Name:         svc-readiness
Namespace:    default
Labels:       <none>
Annotations:  endpoints.kubernetes.io/last-change-trigger-time: 2025-11-27T03:12:34Z
Subsets:
  Addresses:          20.100.194.106
  NotReadyAddresses:  20.110.126.49
  Ports:
    Name     Port  Protocol
    ----     ----  --------
    <unset>  8080  TCP

Events:  <none>
```
   - NotReadyAddresses에 Pod IP가 들어가게 됨
   - ReadinessProbe가 성공하도록 파일 생성 (Pod가 배치된 노드 확인 후 해당 Node에 접속)
```bash
// Pod가 배치된 노드 확인
cd /tmp/readiness
touch ready.txt
```
```bash
Thu Nov 27 12:14:48 KST 2025
Hostname : pod1
Thu Nov 27 12:14:49 KST 2025
Hostname : pod-readiness-exec1
Thu Nov 27 12:14:50 KST 2025
Hostname : pod-readiness-exec1
Thu Nov 27 12:14:51 KST 2025
Hostname : pod-readiness-exec1
Thu Nov 27 12:14:52 KST 2025
Hostname : pod1
Thu Nov 27 12:14:53 KST 2025
Hostname : pod1
Thu Nov 27 12:14:54 KST 2025
Hostname : pod-readiness-exec1
Thu Nov 27 12:14:55 KST 2025
Hostname : pod1
Thu Nov 27 12:14:56 KST 2025
Hostname : pod1
Thu Nov 27 12:14:57 KST 2025
Hostname : pod-readiness-exec1
Thu Nov 27 12:14:58 KST 2025
Hostname : pod-readiness-exec1
Thu Nov 27 12:15:00 KST 2025
Hostname : pod1
Thu Nov 27 12:15:01 KST 2025
Hostname : pod1
Thu Nov 27 12:15:02 KST 2025
Hostname : pod1
Thu Nov 27 12:15:03 KST 2025
Hostname : pod1
Thu Nov 27 12:15:04 KST 2025
Hostname : pod-readiness-exec1
```
   - 3번 성공했으므로, 해당 Pod로도 정상적으로 트래픽이 흐름
   - 상태 확인
```bash
[root@k8s-master ~]# kubectl describe pod pod-readiness-exec1 | grep -A5 Conditions
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
```
```bash
[root@k8s-master ~]# kubectl describe endpoints svc-readiness
Name:         svc-readiness
Namespace:    default
Labels:       <none>
Annotations:  endpoints.kubernetes.io/last-change-trigger-time: 2025-11-27T03:14:15Z
Subsets:
  Addresses:          20.100.194.106,20.110.126.49
  NotReadyAddresses:  <none>
  Ports:
    Name     Port  Protocol
    ----     ----  --------
    <unset>  8080  TCP

Events:  <none>
```

2. LivenessProbe
- App 장애시 헬스체크를 통해 Pod를 Restart 시켜
- initialDelaySeconds : Pod 생성후 해당 지연시간이 지난 후에 헬스체크 시작
- failueThreshold : 실패시 Pod를 Restart 시킴
- successThreshold : 성공시 계속 헬스체크 시도
- periodSeconds : 10초 간격으로 헬스 체크
- Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-liveness
spec:
  selector:
    app: liveness
  ports:
  - port: 8080
    targetPort: 8080
```
  - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  labels:
    app: liveness
spec:
  containers:
  - name: container
    image: kubetm/app
    ports:
    - containerPort: 8080
  terminationGracePeriodSeconds: 0
```

   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-liveness-httpget1
  labels:
    app: liveness
spec:
  containers:
  - name: liveness
    image: kubetm/app
    ports:
    - containerPort: 8080
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
      failureThreshold: 3
  terminationGracePeriodSeconds: 0
```
   - Master Node에서 Service IP로 호출 (1초 단위)
```bash
while true; do date && curl <service-ip>:8080/health; sleep 1; done
```
```bash
[root@k8s-master ~]# while true; do date && curl 10.109.56.69:8080/hostname; sleep 1; done
Thu Nov 27 12:18:10 KST 2025
Hostname : pod-liveness-httpget1
Thu Nov 27 12:18:11 KST 2025
Hostname : pod2
Thu Nov 27 12:18:12 KST 2025
Hostname : pod-liveness-httpget1
Thu Nov 27 12:18:13 KST 2025
Hostname : pod2
Thu Nov 27 12:18:14 KST 2025
Hostname : pod-liveness-httpget1
Thu Nov 27 12:18:15 KST 2025
Hostname : pod-liveness-httpget1
...
```

   - 새 Master Node 탭을 열어서 Pod의 상세 내용 실시간 조회
```bash
// pod-readiness-exec1이름의 Pod 상세 내용중에 | Events와 매칭되는 단어에서 20번째 줄까지 출력
watch "kubectl describe pod pod-liveness-httpget1 | grep -A10 Events"
```
```bash
[root@k8s-master ~]# watch "kubectl describe pod pod-liveness-httpget1 | grep -A10 Events"

Every 2.0s: kubectl describe pod pod-liveness-httpget1 |...  k8s-master: Thu Nov 27 12:23:04 2025

Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  5m41s                default-scheduler  Successfully assigned default/pod-l
iveness-httpget1 to k8s-worker2
  Normal   Pulled     5m37s                kubelet            Successfully pulled image "kubetm/a
pp" in 2.444621215s (2.444632836s including waiting)
```

   - 새 Master Node 탭을 열어서 Pod(pod-liveness-httpget1)를 500(service unavailable)상태로 만듦
```bash
curl <pod-ip>:8080/status500
```
```bash
[root@k8s-master ~]# curl 20.110.126.50:8080/status500
Status Code has Changed to 500
```

   - HTTP Status : 200 ~ 400까지는 성공, 그외의 상태코드는 실패
```bash
[root@k8s-master ~]# watch "kubectl describe pod pod-liveness-httpget1 | grep -A10 Events"

Every 2.0s: kubectl describe pod pod-liveness-httpget1 |...  k8s-master: Thu Nov 27 12:23:47 2025

Events:
  Type     Reason     Age                   From               Message
  ----     ------     ----                  ----               -------
  Normal   Scheduled  6m24s                 default-scheduler  Successfully assigned default/pod-
liveness-httpget1 to k8s-worker2
  Normal   Pulled     6m20s                 kubelet            Successfully pulled image "kubetm/
app" in 2.444621215s (2.444632836s including waiting)
  Warning  Unhealthy  2m4s (x3 over 2m24s)  kubelet            Liveness probe failed: HTTP probe
failed with statuscode: 500
  Normal   Killing    2m4s                  kubelet            Container liveness failed liveness
 probe, will be restarted
  Normal   Pulling    2m1s (x2 over 6m22s)  kubelet            Pulling image "kubetm/app"
  Normal   Created    118s (x2 over 6m20s)  kubelet            Created container liveness
  Normal   Started    118s (x2 over 6m19s)  kubelet            Started container liveness
  Normal   Pulled     118s                  kubelet            Successfully pulled image "kubetm/
app" in 3.03809959s (3.038109029s including waiting)
```
   - 다시 재생성되는 것 확인 가능능

   - 실습 후 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 or Master Node에서 아래 명령 실행)
```bash
kubectl delete pod pod1 pod2 pod-readiness-exec1 pod-liveness-httpget1
kubectl delete svc svc-readiness svc-liveness
```
