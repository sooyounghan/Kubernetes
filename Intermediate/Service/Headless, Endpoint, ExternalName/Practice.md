-----
### Service - Headless, Endpoint, ExternalName
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/44240342-c5fb-4446-bab9-7d8a87ae6cbd">
</div>

1. ClusterIP
   - Service의 이름이 도메인으로 자동 등록되기 때문에, Pod에서는 Service의 이름으로 호출 가능
   - 또한 쿠버네티스에 설치된 CoreDNS를 통해 Service의 이름으로 Serivce IP를 확인 가능
   - Service FQDN : ```<service-name>.<namespace-name>.svc.cluster.local``` (같은 namespace 상에서는 <service-name>만, 타 namespace를 호출시 <service-name>.<namespace-name>까지 입력 필요)
   - Pod FQDN : ```<pod-ip>.<namespace-name>.pod.cluster.local``` (실 사용 불가)
   - Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: clusterip1
spec:
  selector:
    svc: clusterip
  ports:
  - port: 80
    targetPort: 8080
```
   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    svc: clusterip
spec:
  containers:
  - name: container
    image: kubetm/app
```
  - Request Pod
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
  - Master에서 kubectl을 이용해 Pod(request-pod) 내부로 접근
```bash
kubectl exec request-pod -it -- /bin/bash
```
  - nslookup를 통해 DNS 질의
```bash
nslookup clusterip1
nslookup clusterip1.default.svc.cluster.local
```
```bash
[root@k8s-master ~]# kubectl exec request-pod -it -- /bin/bash
[root@request-pod /]# nslookup clusterip1
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	clusterip1.default.svc.cluster.local
Address: 10.103.110.190

[root@request-pod /]# nslookup clusterip1.default.svc.cluster.local
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	clusterip1.default.svc.cluster.local
Address: 10.103.110.190
```

  - Domain 이름을 이용해서 Pod 호출
```bash
curl clusterip1/hostname
curl clusterip1.default.svc.cluster.local/hostname
```
```bash
[root@request-pod /]# curl clusterip1/hostname
Hostname : pod1
[root@request-pod /]# curl clusterip1.default.svc.cluster.local/hostname
Hostname : pod1
```

2. Headless Service
   - Headless Service를 만들면 Service의 IP는 할당되지 않음
   - 그래서 DNS에 Service 호출 시 Service IP는 없고, 해당 Service에 연결된 Pod의 IP들을 반환함
   - 또한 Headless Service를 통해 Pod를 Domain 이름으로 호출 가능
   - Pod FQDN : ```<pod-name>.<service-name>.<namespace-name>.svc.cluster.local```  
   - Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless1
spec:
  selector:
    svc: headless
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
  name: pod4
  labels:
    svc: headless
spec:
  hostname: pod-a
  subdomain: headless1
  containers:
  - name: container
    image: kubetm/app
---
apiVersion: v1
kind: Pod
metadata:
  name: pod5
  labels:
    svc: headless
spec:
  hostname: pod-b
  subdomain: headless1
  containers:
  - name: container
    image: kubetm/app
```
  - Pod(request-pod)에서 nslookup를 통해 DNS 질의
```bash
nslookup headless1
nslookup pod-a.headless1
nslookup pod-b.headless1
```
```bash
[root@request-pod /]# nslookup headless1
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	headless1.default.svc.cluster.local
Address: 20.100.194.118
Name:	headless1.default.svc.cluster.local
Address: 20.110.126.57

[root@request-pod /]# nslookup pod-a.headless1
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	pod-a.headless1.default.svc.cluster.local
Address: 20.110.126.57

[root@request-pod /]# nslookup pod-b.headless1
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	pod-b.headless1.default.svc.cluster.local
Address: 20.100.194.118
```

  - Pod(request-pod)에서 Pod FQDN을 이용해서 Pod 호출
```bash
curl pod-a.headless1:8080/hostname
curl pod-b.headless1:8080/hostname
```
```bash
[root@request-pod /]# curl pod-a.headless1:8080/hostname
Hostname : pod-a
[root@request-pod /]# curl pod-b.headless1:8080/hostname
Hostname : pod-b
```

3. Endpoint - 1/3
   - Service 생성시 Service의 이름과 동일한 이름으로 EndPoint를 만들어줌
   - Endpoint에는 Service에 연결된 Pod의 IP:Port 정보가 관리됨
   - Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: endpoint1
spec:
  selector:
    svc: endpoint
  ports:
  - port: 8080
```
  - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod7
  labels:
    svc: endpoint
spec:
  containers:
  - name: container
    image: kubetm/app
```
   - Master에서 endpoint 조회
```bash
kubectl describe endpoints endpoint1
```
```bash
[root@k8s-master ~]# kubectl describe endpoints endpoint1
Name:         endpoint1
Namespace:    default
Labels:       <none>
Annotations:  endpoints.kubernetes.io/last-change-trigger-time: 2025-11-27T08:04:58Z
Subsets:
  Addresses:          20.110.126.58
  NotReadyAddresses:  <none>
  Ports:
    Name     Port  Protocol
    ----     ----  --------
    <unset>  8080  TCP

Events:  <none>
```

4. Endpoint - 2/3
  - Endpoint를 사용자가 직접 만들 수 있음. 이 경우 Pod의 IP:Port 정보도 수동 입력 필요
  - Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: endpoint2
spec:
  ports:
  - port: 8080
```
  - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod9
spec:
  containers:
  - name: container
    image: kubetm/app
```
  - Endpoint
```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: endpoint2
subsets:
 - addresses:
   - ip: <pod-ip>
   ports:
   - port: 8080
```
  - Pod(request-pod)에서 Service(endpoint2)로 연결된 Pod 호출
```bash
kubectl exec request-pod -it -- /bin/bash
curl endpoint2:8080/hostname
```
```bash
[root@k8s-master ~]# kubectl exec request-pod -it -- /bin/bash
[root@request-pod /]# curl endpoint2:8080/hostname
Hostname : pod9
```

5. Endpoint - 3/3
   - Endpoint를 사용자가 직접 만들 때 외부 연결 IP를 설정할 수 있음
   - Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: endpoint3
spec:
  ports:
  - port: 443
```
   - Github를 IP로 호출 : GitHub 보안강화로 Host를 Header 넣어야 다운로드가 가능해짐
```bash
nslookup raw.githubusercontent.com
curl -k -O https://185.199.110.133/k8s-1pro/install/refs/heads/main/container/daeku/app/app.js -H "Host: raw.githubusercontent.com"
```
```bash
[root@request-pod /]# nslookup raw.githubusercontent.com
Server:		10.96.0.10
Address:	10.96.0.10#53

Non-authoritative answer:
Name:	raw.githubusercontent.com
Address: 185.199.108.133
Name:	raw.githubusercontent.com
Address: 185.199.110.133
Name:	raw.githubusercontent.com
Address: 185.199.109.133
Name:	raw.githubusercontent.com
Address: 185.199.111.133
Name:	raw.githubusercontent.com
Address: 2606:50c0:8001::154
Name:	raw.githubusercontent.com
Address: 2606:50c0:8003::154
Name:	raw.githubusercontent.com
Address: 2606:50c0:8002::154
Name:	raw.githubusercontent.com
Address: 2606:50c0:8000::154

[root@request-pod /]# curl -k -O https://185.199.110.133/k8s-1pro/install/refs/heads/main/container/daeku/app/app.js -H "Host: raw.githubusercontent.com"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2006  100  2006    0     0   3674      0 --:--:-- --:--:-- --:--:--  3673
```

   - Endpoint
```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: endpoint3
subsets:
 - addresses:
   - ip: 185.199.110.133
   ports:
   - port: 443
```
   - Github에 IP로 호출
```bash
curl -k -O https://endpoint3/k8s-1pro/install/refs/heads/main/container/daeku/app/app.js -H "Host: raw.githubusercontent.com"
```
```bash
curl -k -O https://endpoint3/k8s-1pro/install/refs/heads/main/container/daeku/app/app.js -H "Host: raw.githubusercontent.com"
```

6. ExternalName
   - Pod에서 외부 도메인을 호출할 때 외부 도메인을 Service로 등록해 놓으면, 추후 외부 도메인 주소가 바꼈을 때 Service의 수정만으로 해결 가능 (Service가 없었다면 Pod를 수정하고 재시작 시켰어야 함)
   - Service
```yaml
apiVersion: v1
kind: Service
metadata:
 name: externalname1
spec:
 type: ExternalName
 externalName: raw.githubusercontent.com # 도메인 이름이 바뀌게 되면, 서비스의 externalName에서 해당 도메인 이름만 변경함녀 되므로 Pod를 재배포하지 않아도 됨
```
   - externalname1으로 호출
```bash
curl -k -O https://externalname1/k8s-1pro/install/refs/heads/main/container/daeku/app/app.js -H "Host: raw.githubusercontent.com"
```
```bash
curl -k -O https://externalname1/k8s-1pro/install/refs/heads/main/container/daeku/app/app.js -H "Host: raw.githubusercontent.com"
```
  
  - 실습 후 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 or Master Node에서 아래 명령 실행)
```bash
kubectl delete pod pod1 request-pod pod4 pod5 pod7 pod9
kubectl delete svc clusterip1 headless1 endpoint1 endpoint2 endpoint3 externalname1
```

7. DNS
<div align="center">
<img src="https://github.com/user-attachments/assets/82f158ec-e846-4cde-aa07-eebc910dc587">
</div>

8. Headless
<div align="center">
<img src="https://github.com/user-attachments/assets/8499f65f-55c7-42ab-978b-da9ff74ceb0e">
</div>

9. External
<div align="center">
<img src="https://github.com/user-attachments/assets/829b1654-65ec-48b1-bb87-040198b60053">
</div>
