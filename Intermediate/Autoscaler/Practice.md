-----
### AutoScaler - HPA
-----
<img width="885" height="232" alt="image" src="https://github.com/user-attachments/assets/5f078a9a-568e-4d35-aeb9-f09290080eea" />

1. Metrics Server 설치 (쿠버네티스 클러스터 구축시 함께 설치됨)
   -  Metrics Server 설치 확인
```bash
kubectl get pod -n kube-system
kubectl get apiservices | egrep metrics

// Node 메트릭 확인
kubectl top node
```
```bash
[root@k8s-master ~]# kubectl get pod -n kube-system
NAME                                 READY   STATUS    RESTARTS       AGE
coredns-5d78c9869d-gj947             1/1     Running   14 (87m ago)   3d13h
coredns-5d78c9869d-qvks6             1/1     Running   14 (87m ago)   3d13h
etcd-k8s-master                      1/1     Running   14 (87m ago)   3d13h
kube-apiserver-k8s-master            1/1     Running   14 (87m ago)   3d13h
kube-controller-manager-k8s-master   1/1     Running   32 (87m ago)   3d13h
kube-proxy-fk6b5                     1/1     Running   13 (61m ago)   3d13h
kube-proxy-hrcvc                     1/1     Running   9 (87m ago)    3d13h
kube-proxy-qkm8h                     1/1     Running   14 (87m ago)   3d13h
kube-scheduler-k8s-master            1/1     Running   30 (87m ago)   3d13h
metrics-server-7db4fb59f9-7w7d2      1/1     Running   14 (87m ago)   3d13h

[root@k8s-master ~]# kubectl get apiservices | egrep metrics
v1beta1.metrics.k8s.io                 kube-system/metrics-server    True        3d13h

[root@k8s-master ~]# kubectl top node
NAME          CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
k8s-master    2706m        67%    2552Mi          67%       
k8s-worker1   644m         21%    1160Mi          40%       
k8s-worker2   596m         19%    1473Mi          51%
```

​2. HPA (Horizontal Pod Autoscaler) 
   - Target Deployment (CPU) / Service 
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: stateless-cpu1
spec:
 selector:
   matchLabels:
      resource: cpu
 replicas: 2
 template:
   metadata:
     labels:
       resource: cpu
   spec:
     containers:
     - name: container
       image: kubetm/app:v1
       resources:
         requests:
           cpu: 10m
         limits:
           cpu: 20m
---
apiVersion: v1
kind: Service
metadata:
 name: stateless-svc1
spec:
 selector:
    resource: cpu
 ports:
   - port: 8080
     targetPort: 8080
     nodePort: 30001
 type: NodePort
```

   - HPA - Resource (Utilization) 
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-resource-cpu
spec:
  maxReplicas: 10
  minReplicas: 2
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: stateless-cpu1
  metrics:
  - type: Resource 
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```
   -  HPA 모니터링
```bash
kubectl get hpa -w
```

   - 부하 주입
```bash
while true; do curl 192.168.56.30:30001/hostname; sleep 0.05; done
```

   - 다음 실습 모니터링을 위해 HPA 삭제
```bash
kubectl delete hpa hpa-resource-cpu
```

  - Target Deployment (Memory) / Service
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: stateless-memory1
spec:
 selector:
   matchLabels:
      resource: memory
 replicas: 2
 template:
   metadata:
     labels:
       resource: memory
   spec:
     containers:
     - name: container
       image: kubetm/app:v1
       resources:
         requests:
           memory: 5Mi
         limits:
           memory: 20Mi
---
apiVersion: v1
kind: Service
metadata:
 name: stateless-svc2
spec:
 selector:
    resource: memory
 ports:
   - port: 8080
     targetPort: 8080
     nodePort: 30002
 type: NodePort
```

   - HPA - Resource (AverageValue) 
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-resource-memory
spec:
  maxReplicas: 10
  minReplicas: 2
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: stateless-memory1
  metrics:
  - type: Resource 
    resource:
      name: memory
      target:
        type: AverageValue
        averageValue: 5Mi
```
  - 부하 주입
```bash
while true;do curl 192.168.56.30:30002/hostname; sleep 0.1; done
```

  - 실습 후 모든 리소스 삭제 ​(Dashboard에서 리소스별 삭제 or Master Node에서 아래 명령 실행)
```bash
kubectl delete hpa hpa-resource-cpu hpa-resource-memory
kubectl delete deploy stateless-cpu1 stateless-memory1
kubectl delete svc stateless-svc1 stateless-svc2
```
