-----
### 실습 - Namespace, ResourceQuota, LimitRange
-----
<img width="884" height="230" alt="image" src="https://github.com/user-attachments/assets/8b533cbc-6402-4472-bbe3-5c2d2ea8ddfb" />

1. Namespace
  - Namespace는 App(Pod + Service)간에 논리적인 구분을 하기 위해 그룹과 같은 개념으로 사용
  - Namespace 내에서 같은 종류의 Object는 이름이 동일할 수 없음  
  - Service의 Selector로 Pod를 연결시 같은 Namespace 상에서만 연결 가능
  - Pod간의 네트워크 통신이나, Node를 이용한 Volume은 Namespace가 서로 다르더라도 연결 및 공유 가능
  - Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-1
```
   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: nm-1 # 네임스페이스 지정
  labels:
    app: pod
spec:
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
  namespace: nm-1 # 네임스페이스 지정
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080
```
   - Namespace 2
```yaml 
apiVersion: v1
kind: Namespace
metadata:
  name: nm-2
```
   - Service 2
     + 타 Namespace의 Pod와 연결이 되는지 확인하기
     + Namespace가 다를 경우 Object 이름이 중복 가능한지 확인하
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-1
  namespace: nm-2
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080
```
   - Pod 2
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: nm-2
  labels:
    app: pod
spec:
  containers:
  - name: container
    image: kubetm/app
    ports:
    - containerPort: 8080
```
   - Namespace(nm-2) : Pod(pod-1)에 들어가서 타 Namespace(nm-1)에 있는 Pod(pod-2) 호출
```bash
// Pod IP
curl <pod-ip>:8080/hostname

// Service
curl <service-ip>:9000/hostname
```
  - ```<pod-ip>, <service-ip>```는 본인의 Namesapce(nm-1)의 Pod와 Service에 만들어진 IP
```bash
root@pod-1:/# curl 20.100.194.83:8080/hostname
Hostname : pod-1
root@pod-1:/# curl 10.96.58.2:9000/hostname
Hostname : pod-1
```

2. Namespace로 분리되지 않는 자원 테스트
   - NodePort : 분리되지 않음
     + Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-2
  namespace: nm-1
spec:
  ports:
  - port: 9000
    targetPort: 8080
    nodePort: 30001 # 30000번대 포트
  type: NodePort
---
apiVersion: v1
kind: Service
metadata:
  name: svc-2
  namespace: nm-2
spec:
  ports:
  - port: 9000
    targetPort: 8080
    nodePort: 30002
  type: NodePort
```
   - kubernetes dahsboard UI Port가 30000라서 실습 port는 30001로 설정
   - Volume (Node) : 공유유
     + Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
  namespace: nm-1
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-worker1
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: host-path
      mountPath: /mount1
  volumes:
  - name : host-path
    hostPath:
      path: /node-v
      type: DirectoryOrCreate
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
  namespace: nm-2
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-worker1
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: host-path
      mountPath: /mount1
  volumes:
  - name : host-path
    hostPath:
      path: /node-v
      type: DirectoryOrCreate
```
   - Namespace(nm-1) : Pod(pod-2)에 들어가서 파일 생성
```bash
cd /mount1
echo "hello" >> hello.txt
ls
```
```bash
[root@pod-2 /]# cd /mount1
[root@pod-2 mount1]# echo "hello" >> hello.txt
[root@pod-2 mount1]# ls
file.txt  hello.txt
```

   - Namespace(nm-2) : Pod(pod-2)에 들어가서 파일 공유 확인
```bash
cd /mount1
ls
```
```bash
[root@pod-2 /]# cd /mount1
[root@pod-2 mount1]# ls
file.txt  hello.txt
```

  - 실습 후 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 또는 Master Node에서 아래 명령 실행)
```bash
kubectl delete ns nm-1 nm-2
```
  - Namespace 삭제시 소속된 모든 Object들은 자동 삭제

2. ResourceQuota
   - Cluster나 Namespace별 자원사용량을 제한시키 위해 사용 (실무에서는 관리가 힘들어서 잘 사용되지 않음)
   - 특정 Namespace에서 ResourceQuota 설정시 해당 Namespace에 만들어지는 Pod에 Resource 설정 필수
   - ResourceQuota로 제한되는 항목은 Docs 확인 필요 (```https://kubernetes.io/ko/docs/concepts/policy/resource-quotas/```)
   - Namespace
 ```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-3
```
   - ResourceQuota
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-1
  namespace: nm-3
spec:
  hard:
    requests.memory: 1Gi # Request Memory 1GB 지정
    limits.memory: 1Gi # Limit Memory 1GB 지정
```
   - Master Node에서 kubectl 명령으로 확인
```bash
kubectl describe resourcequotas --namespace=nm-3
```
```bash
[root@k8s-master ~]# kubectl describe resourcequotas --namespace=nm-3
Name:            rq-1
Namespace:       nm-3
Resource         Used  Hard
--------         ----  ----
limits.memory    0     1Gi
requests.memory  0     1Gi
```

   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
  namespace: nm-3
spec:
  containers:
  - name: container
    image: kubetm/app
```
   - resoucres 부분 제외 :Dashboard에서 아래와 같은 에러 발생 
```bash
Deploying file has failed
pods "pod-2" is forbidden: failed quota: rq-1: must specify limits.memory for: container; requests.memory for: container
```

   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
  namespace: nm-3
spec:
  containers:
  - name: container
    image: kubetm/app
    resources: # 반드시 설정
      requests: 
        memory: 0.5Gi
      limits:
        memory: 0.5Gi
```
   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-4
  namespace: nm-3
spec:
  containers:
  - name: container
    image: kubetm/app
    resources:
      requests:
        memory: 0.8Gi
      limits:
        memory: 0.8Gi
```
   - 1GB가 넘으므로 Dashboard에서 아래와 같은 에러 발생
```bash
Deploying file has failed
pods "pod-4" is forbidden: exceeded quota: rq-1, requested: limits.memory=858993459200m,requests.memory=858993459200m, used: limits.memory=512Mi,requests.memory=512Mi, limited: limits.memory=1Gi,requests.memory=1Gi
```
   - ResourceQuota
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-2
  namespace: nm-3
spec:
  hard:
    pods: 2 # Pod 개수 제한
```
   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-4
  namespace: nm-3
spec:
  containers:
  - name: container
    image: kubetm/app
    resources:
      requests:
        memory: 0.1Gi
      limits:
        memory: 0.1Gi
```
   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-5
  namespace: nm-3
spec:
  containers:
  - name: container
    image: kubetm/app
    resources:
      requests:
        memory: 0.1Gi
      limits:
        memory: 0.1Gi
```
   - 2개 사용 중 : Dashboard에서 아래와 같은 에러 발생
```bash
Deploying file has failed
pods "pod-5" is forbidden: exceeded quota: rq-2, requested: pods=1, used: pods=2, limited: pods=2
```

   - ResourceQuota의 예외 상황 테스트 : ResourceQutoa가 명시되어 있지 않는 Pod가 있는 상태에서 ResourceQuota를 만들 경우, 기존 Pod는 ResourceQuota 규칙에서 벗어나는 상황이 발생
     + Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-4
```
   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
  namespace: nm-4
spec:
  containers:
  - name: container
    image: kubetm/app
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
  namespace: nm-4
spec:
  containers:
  - name: container
    image: kubetm/app
```
   - ResourceQuota
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-1
  namespace: nm-4
spec:
  hard:
    requests.memory: 1Gi
    limits.memory: 1Gi
```
   - 기존의 Pod를 유지한 채 생성

   - Pod : ResourceQuota 설정 이후 그 값이 적용되는 것이므로 2개가 Pod가 차있지만, 설정 기준 이후로는 1개 추가된 것이므로 넣어지므로 총 3개 Pod 존재
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-5
  namespace: nm-4
spec:
  containers:
  - name: container
    image: kubetm/app
    resources:
      requests:
        memory: 1Gi
      limits:
        memory: 1Gi
```
   - 즉, Namespace에 지정된 ResourceQuota에 명시도니 것보다 더 많은 양의 자원 사용
     + 주의사항 : ResourceQuota가 만들어지기전에 해당 Namespace에 Pod가 존재하지 않도록 해야함
   - 실습 후 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 또는 Master Node에서 아래 명령 실행)
```bash
kubectl delete ns nm-3 nm-4
```
   - Namespace 삭제시 소속된 모든 Object들은 자동 삭제

3. LimitRange
  - Cluster나 Namespace별 자원사용량을 제한시키 위해 사용 (실무에서는 관리가 힘들어서 잘 사용되진 않음)
  - LimitRange는 나의 Namespace 들어올 수 있는 Pod의 자원을 제한 시키는 역할
  - Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-5
```
   - LimitRange
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: lr-1
  namespace: nm-5
spec:
  limits:
  - type: Container
    min:
      memory: 0.1Gi
    max:
      memory: 0.4Gi
    maxLimitRequestRatio:
      memory: 3
    defaultRequest:
      memory: 0.1Gi
    default:
      memory: 0.2Gi
```
   - Master Node에서 kubectl 명령으로 확인
```bash
kubectl describe limitranges --namespace=nm-5
```
```bash
[root@k8s-master ~]# kubectl describe limitranges --namespace=nm-5
Name:       lr-1
Namespace:  nm-5
Type        Resource  Min            Max            Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---            ---            ---------------  -------------  -----------------------
Container   memory    107374182400m  429496729600m  107374182400m    214748364800m  3
```

   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: nm-5
spec:
  containers:
  - name: container
    image: kubetm/app
    resources:
      requests:
        memory: 0.1Gi
      limits:
        memory: 0.5Gi # maxLimitRequestRatio 1 : 5
```
   - Dashboard에서 아래와 같은 에러 발생
```bash
Deploying file has failed
pods "pod-1" is forbidden: [maximum memory usage per Container is 429496729600m, but limit is 512Mi, memory max limit to request ratio per Container is 3, but provided ratio is 5.000000]
```

   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: nm-5
spec:
  containers:
  - name: container
    image: kubetm/app
    resources:
      requests:
        memory: 0.2Gi
      limits:
        memory: 0.4Gi
```
   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
  namespace: nm-5
spec:
  containers:
  - name: container
    image: kubetm/app
```
   - ResourceQuota의 예외 상황 테스트
     + Resource가 명시되어 있지 않는 Pod가 있는 상태에서 ResourceQuota를 만들 경우, 기존 Pod는 ResourceQuota 규칙에서 벗어나는 상황이 발생
   - Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-6
```
  - LimitRange
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: lr-5
  namespace: nm-6
spec:
  limits:
  - type: Container
    min:
      memory: 0.1Gi
    max:
      memory: 0.5Gi # 최대 0.5 GB
    maxLimitRequestRatio:
      memory: 1 # 비율 동일
    defaultRequest:
      memory: 0.5Gi
    default:
      memory: 0.5Gi
```
   - LimitRange
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: lr-3
  namespace: nm-6
spec:
  limits:
  - type: Container
    min:
      memory: 0.1Gi
    max:
      memory: 0.3Gi # 최대 0.3GB
    maxLimitRequestRatio:
      memory: 1 # 비율은 동일
    defaultRequest:
      memory: 0.3Gi
    default:
      memory: 0.3Gi
```
   - Master Node에서 kubectl 명령으로 확인
```bash
kubectl describe limitranges --namespace=nm-6
```
```bash
[root@k8s-master ~]# kubectl describe limitranges --namespace=nm-6
Name:       lr-3
Namespace:  nm-6
Type        Resource  Min            Max            Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---            ---            ---------------  -------------  -----------------------
Container   memory    107374182400m  322122547200m  322122547200m    322122547200m  1


Name:       lr-5
Namespace:  nm-6
Type        Resource  Min            Max    Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---            ---    ---------------  -------------  -----------------------
Container   memory    107374182400m  512Mi  512Mi            512Mi          1
[root@k8s-master ~]# kubectl describe limitranges --namespace=nm-6
Name:       lr-3
Namespace:  nm-6
Type        Resource  Min            Max            Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---            ---            ---------------  -------------  -----------------------
Container   memory    107374182400m  322122547200m  322122547200m    322122547200m  1


Name:       lr-5
Namespace:  nm-6
Type        Resource  Min            Max    Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---            ---    ---------------  -------------  -----------------------
Container   memory    107374182400m  512Mi  512Mi            512Mi          1
```

  - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: nm-6
spec:
  containers:
  - name: container
    image: kubetm/app
```
   - Dashboard에서 아래와 같은 에러 발생
     + 설정 LimitRange 2개 (0.3 GB, 0.5 GB) 중 더 적은 값으로 제한
     + Pod 생성 시 limit 값을 명시하지 않으면 default 값으로 생성되려하는데, 큰 Default로 잡힘
     + 즉, 한 네임스페이스에 여러 개의 LimitRange를 지정하게 되면 Pod 생성이 안 될 수 있으므로 주의
```bash
Deploying file has failed
pods "pod-1" is forbidden: maximum memory usage per Container is 322122547200m, but limit is 512Mi
```

   - 실습 후 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 or Master Node에서 아래 명령 실행)
```bash
kubectl delete ns nm-5 nm-6
```
​   - Namespace 삭제시 소속된 모든 Object들은 자동 삭제
