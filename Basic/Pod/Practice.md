-----
### Pod 실습
-----
1. Pod : 한 Pod에 여러 컨테이너를 생성하는 예제 (Dashboard 상단위 + 버튼으로 추가)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers: # 두 개의 컨테이너 생성
  - name: container1
    image: kubetm/p8000 # 이미지 이름은 p8000
    ports:
    - containerPort: 8000
  - name: container2
    image: kubetm/p8080 # 이미지 이름은 p8080
    ports:
    - containerPort: 8080
```
   - Pod 생성 시 확인사항
     + Kubernetes Dashboard 상단 콤보박스에서 꼭 Namespace를 [default]로 해서 작업
     + 만약 [모든 네임스페이스]로 되어 있다면 Pod 생성시 [Deploying file has failed] 에러가 발생
<div align="center">
<img src="https://github.com/user-attachments/assets/8403f067-666d-4169-8dbc-a128dd4d6014">
</div>

   - 네트워크의 IP는 서비스 없이 이 Pod에 접근할 수 있는 IP이고, Kubernetes Cluster 내에서만 접근 가능
   - Master Node에서 Pod-IP:Port로 각각 다른 Container 호출 (XShell)
```bash
curl <pod-ip>:8000
curl <pod-ip>:8080
```
```
[root@k8s-master ~]# curl 20.108.82.212:8000
containerPort : 8000
[root@k8s-master ~]# curl 20.108.82.212:8080
containerPort : 8080
```

   - Container 내부에서 들어가서 Pod 내부의 다른 Container 호출
<div align="center">
<img src="https://github.com/user-attachments/assets/95e94c37-6a96-46ba-9f38-84d89b9f0293">
</div>

```bash
curl localhost:8000
curl localhost:8080
```
<div align="center">
<img src="https://github.com/user-attachments/assets/54dc7077-6e5f-4789-a903-69d9600915b8">
</div>

   - Pod 생성 시, Container Port를 중복시킬 경우
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers: # 두 개의 컨테이너 생성
  - name: container1
    image: kubetm/p8000 
    ports:
    - containerPort: 8000
  - name: container2
    image: kubetm/p8000
    ports:
    - containerPort: 8000
```
<div align="center">
<img src="https://github.com/user-attachments/assets/6f7133dd-2232-4838-9e43-0b9850d2876a">
</div>

   - 현재 8000번 Port가 이미 사용 : Conatiner 1을 만들고, Container를 만들면서 1에서 이미 8000번 Port를 사용했기 때문임
   - 즉, 한 Pod 내에 있는 컨테이너들끼리는 서로 Port 중복이 될 수 없음

   - Pod가 시스템에 의해서 관리가 됐을 때, Pod의 IP가 재생성되면 변경되는 경우
     + 컨트롤러 생성
```yaml
apiVersion: v1
kind: ReplicationController # 컨트롤러 : Pod 생성 및 죽었을 때 재생성시켜주는 관리 역할
metadata:
  name: replication-1
spec:
  replicas: 1
  selector:
    app: rc
  template:
    metadata:
      name: pod-1
      labels:
        app: rc
    spec:
      containers:
      - name: container
        image: tmkube/init
```
<div align="center">
<img src="https://github.com/user-attachments/assets/24bd9558-b5fd-45c0-9851-6d1d993f2ba9">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/2c0e1fe8-1757-48b0-8145-8151109414b8">
</div>

2. Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deploy
  template:
    metadata:
      name: pod-1
      labels:
        app: deploy
    spec:
      containers:
      - name: container
        image: kubetm/init
```
   - 실습 후 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 버튼 또는 Master Node에서 아래 명령 실행)
```bash
kubectl delete pod pod-1 
kubectl delete deploy deployment-1
```

3. Label
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1 # 개발 환경에서 실핼될 웹 서버
  labels:
    type: web
    lo: dev
spec:
  containers:
  - name: container
    image: kubetm/init
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-2 # 개발 환경에서 만들 DB 서버
  labels:
    type: db
    lo: dev
spec:
  containers:
  - name: container
    image: kubetm/init
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
  labels:
    type: server
    lo: dev
spec:
  containers:
  - name: container
    image: kubetm/init
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-4
  labels:
    type: web
    lo: production
spec:
  containers:
  - name: container
    image: kubetm/init
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-5
  labels:
    type: db
    lo: production
spec:
  containers:
  - name: container
    image: kubetm/init
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-6
  labels:
    type: server
    lo: production
spec:
  containers:
  - name: container
    image: kubetm/init
```
<div align="center">
<img src="https://github.com/user-attachments/assets/129c117b-25d0-4c34-b927-39f2e190ceaa">
</div>

4. Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-for-web # 원하는 파드들을 선택 : web
spec:
  selector:
    type: web
  ports:
  - port: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: svc-for-production # production
spec:
  selector:
    lo: production 
  ports:
  - port: 8080
```
   - Pod에 Label을 설정하면, 해당 목적에 따라 Service로 연결 가능
   - 실습 후 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 버튼 또는 Master Node에서 아래 명령 실행)
```bash
kubectl delete pod pod-1 pod-2 pod-3 pod-4 pod-5 pod-6
kubectl delete svc svc-for-web svc-for-production
```

5. Node Schedule
   - Pod
```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
spec:
  nodeSelector: 
    kubernetes.io/hostname: k8s-worker1 # Kubernetes Host가 노드 1번을 가리킴
  containers:
  - name: container
    image: kubetm/init
```
<div align="center">
<img src="https://github.com/user-attachments/assets/b0a8f6f3-f020-4655-a030-7cca16b191e6">
</div>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-4
spec:
  containers:
  - name: container
    image: kubetm/init
    resources:
      requests:
        memory: 2Gi # 요청 메모리 2GB
      limits:
        memory: 3Gi 
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-5
spec:
  containers:
  - name: container
    image: kubetm/init
    resources:
      requests:
        memory: 0.5Gi # 요청 메모리 0.5GB
      limits:
        memory: 0.5Gi
```
   - Kubernetes는 Scheduling 할 때, 노드마다 일종의 점수를 매겨 가장 높은 점수에 있는 노드에 Pod를 할당
   - 실습 후 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 버튼 또는 Master Node에서 아래 명령 실행)
```bash
kubectl delete pod pod-3 pod-4 pod-5
```

6. Pod Yaml 스펙
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-4                           # Pod 이름
  labels:                               # Label 
    type: web                           
    lo: dev  
spec:
  nodeSelector:                         # Node 직접 지정시
    kubernetes.io/hostname: k8s-worker1
  containers:
  - name: container                     # Container 이름
    image: kubetm/init                  # 이미지 선택
    ports:
    - containerPort: 8080               
    resources:                          # 자원 사용량 설정
      requests:
        memory: 1Gi
      limits:
        memory: 1Gi
```

7. Kubectl 명령어
   - Create
```bash
# pod.yaml 파일이 있을 경우
kubectl create -f ./pod.yaml

# 내용과 함께 바로 작성
kubectl create -f - <<END
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: container
    image: kubetm/init
END
```

   - Apply
```bash
kubectl apply -f ./pod.yaml
```

   - Apply vs Create : 둘다 자원을 생성할때 사용할 수 있지만, [Create]는 기존에 같은 이름의 Pod가 존재하면 생성이 안되고, [Apply]는 기존에 같은 이름의 Pod가 존재하면 업데이트

   - Get
```bash
// 기본 Pod 리스트 조회 (Namepsace 포함)
kubectl get pods -n defalut

// 좀 더 많은 내용 출력
kubectl get pods -o wide

// Pod 이름 지정
kubectl get pod pod1

// Json 형태로 출력
kubectl get pod pod1 -o json
```

   - Describe
```bash
kubectl describe pod pod1
```

   - Delete
```bash
# 파일이 있을 경우 생성한 방법 그대로 삭제
kubectl delete -f ./pod.yaml

# 내용과 함께 바로 작성한 경우 생성한 방법 그대로 삭제
kubectl delete -f - <<END
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: container
    image: kubetm/init
END

# Pod 이름 지정
kubectl delete pod pod1
```

   - Exec
```bash
# Pod이름이 pod1인 Container로 들어가기 (나올땐 exit)
kubectl exec pod1 -it /bin/bash

# Container가 두개 이상 있을때 Container이름이 con1인 Container로 들어가기 
kubectl exec pod1 -c con1 -it /bin/bash
```
