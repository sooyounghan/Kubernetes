-----
### Service - 실습
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/e8d2e5eb-a093-43c0-8ebc-c71cc7e9b4c0">
</div>

1. ClusterIP
   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  labels:
     app: pod
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-worker1
  containers:
  - name: container
    image: kubetm/app
    ports:
    - containerPort: 8080
```
   - Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-1
spec:
  selector: # Pod를 지정하는 selector
    app: pod
  ports:
  - port: 9000 # 9000번으로 접근할 때,
    targetPort: 8080 # 서비스의 타켓이 되는 컨테이너의 8080 포트로 연결
  # type: ClusterIP # 클러스터 IP는 생략 가능
```

   - Master Node에서 Service IP:Port 호출 (Service-IP : Service 생성 후 만들어진 IP를 확인)
```bash
curl <Service-IP>:9000/hostname
```
```bash
[root@k8s-master ~]# curl 10.108.63.239:9000/hostname
Hostname : pod-1
```

   - Pod를 재생성하더라도, Service에 Label과 Selector가 있으므로 연결을 맺어주므로 해당 Service에 연결 (물론 Pod의 IP는 변경되지만 Service는 변경되지 않음)
     
2. NodePort
   - Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-2
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080
    nodePort: 30001 # NodePort 번호
  type: NodePort # 타입은 nodePort
```
   - Kubernetes dahsboard UI Port가 30000라서 Port는 30001로 변경
   - 두 개의 엔드포인트 생성 (svc-2:9000 TCP는 클러스터 IP로 접근했을 때 연결할 IP, svc-2:30001 TCP는 Port가 노드로 접근했을 때 사용하는 IP)
<div align="center">
<img src="https://github.com/user-attachments/assets/87c242fe-f765-4530-98b0-8dee0a722a5d">
</div>

   - cmd : ```<VM IP>:<NodePort>```로 호출하여 외부 연동 확인
```bash
curl 192.168.56.31:30001/hostname
curl 192.168.56.32:30001/hostname
```
```bash
C:\Users\young>curl 192.168.56.31:30001/hostname
Hostname : pod-1

C:\Users\young>curl 192.168.56.32:30001/hostname
Hostname : pod-1
```

   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
  labels:
     app: pod
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-worker2
  containers:
  - name: container
    image: kubetm/app
    ports:
    - containerPort: 8080
```

  - cmd에서 두 Pod에 트래픽 분산 확인
```bash
curl 192.168.56.31:30001/hostname
```
```bash
C:\Users\young>curl 192.168.56.32:30001/hostname
Hostname : pod-1

C:\Users\young>curl 192.168.56.31:30001/hostname
Hostname : pod-1

C:\Users\young>curl 192.168.56.31:30001/hostname
Hostname : pod-2

C:\Users\young>curl 192.168.56.31:30001/hostname
Hostname : pod-2

C:\Users\young>curl 192.168.56.31:30001/hostname
Hostname : pod-1
```

  - Service를 추가하여 externalTrafficPolicy 기능 확인
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-3
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080
    nodePort: 30002
  type: NodePort
  externalTrafficPolicy: Local 
```
   -  cmd : 각 노드별 IP에 따라, 해당 노드 위에 있는 Pod를 호출하게 됨 (노드 1에는 Pod-1, 노드 2에는 Pod-2가 존재)
```bash
curl 192.168.56.31:30002/hostname
curl 192.168.56.32:30002/hostname
```
```bash
C:\Users\young>curl 192.168.56.31:30001/hostname
Hostname : pod-1

C:\Users\young>curl 192.168.56.31:30002/hostname
Hostname : pod-1

C:\Users\young>curl 192.168.56.31:30002/hostname
Hostname : pod-1

C:\Users\young>curl 192.168.56.31:30002/hostname
Hostname : pod-1

C:\Users\young>curl 192.168.56.31:30002/hostname
Hostname : pod-1

C:\Users\young>curl 192.168.56.32:30002/hostname
Hostname : pod-2

C:\Users\young>curl 192.168.56.32:30002/hostname
Hostname : pod-2

C:\Users\young>curl 192.168.56.32:30002/hostname
Hostname : pod-2

C:\Users\young>curl 192.168.56.32:30002/hostname
Hostname : pod-2
```

   - 노드 1에 Pod가 없는데, 노드 1의 IP를 계속 호출할 경우 : 계속 기다리게 되므로 주의해야 함 (DaemonSet과 주로 사용)

3. LoadBalancer
   - Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-4
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080
  type: LoadBalancer
```
   - Pending 상태 진입 : 외부에서 접근할 External IP를 할당해주는 플러그인이 제공되어야 하는데, Kubernetes는 없음
<div align="center">
<img src="https://github.com/user-attachments/assets/6944521d-4ef8-4d1a-aa3e-1640473d93c9">
</div>

   - Master Node에서 명령어 실행
```bash
kubectl get service svc-4
```
```bash
[root@k8s-master ~]# kubectl get service svc-4
NAME    TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
svc-4   LoadBalancer   10.100.112.80   <pending>     9000:31612/TCP   2m37s
```

   - 실습 후 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 또는 Master Node에서 아래 명령 실행)
```bash
kubectl delete pod pod-1 pod-2
kubectl delete svc svc-1 svc-2 svc-3 svc-4
```
