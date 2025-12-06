-----
### 모니터링 설치 - Loki-Stack
-----
1. 해당 설치는 Storage 연동을 따로 할당하지 않았기 때문에 VM이 재기동 될때, 기존 저장된 로그 데이터는 사라짐
2. Github(k8s-1pro)에서 Prometheus(with Grafana), Loki-Stack yaml 다운로드
    - [k8s-master] Console 접속 후 아래 명령 실행
```bash
[root@k8s-master ~]# yum -y install git

# 로컬 저장소 생성
git init monitoring
git config --global init.defaultBranch main
cd monitoring

# remote 추가 ([root@k8s-master monitoring]#)
git remote add -f origin https://github.com/k8s-1pro/install.git

# sparse checkout 설정
git config core.sparseCheckout true
echo "ground/k8s-1.27/prometheus-2.44.0" >> .git/info/sparse-checkout
echo "ground/k8s-1.27/loki-stack-2.6.1" >> .git/info/sparse-checkout

# 다운로드 
git pull origin main
```

3. Prometheus(with Grafana) 설치 (Github : ```https://github.com/prometheus-operator/kube-prometheus/tree/release-0.14```)
```bash
# 설치 ([root@k8s-master monitoring]#)
kubectl apply --server-side -f ground/k8s-1.27/prometheus-2.44.0/manifests/setup
kubectl wait --for condition=Established --all CustomResourceDefinition --namespace=monitoring
kubectl apply -f ground/k8s-1.27/prometheus-2.44.0/manifests

# 설치 확인 ([root@k8s-master]#) 
kubectl get pods -n monitoring
```
<div align="center">
<img src="https://github.com/user-attachments/assets/5331e0bc-e164-443e-b058-77f565653c25">
</div>

4. Loki-Stack 설치
```bash
# 설치 ([root@k8s-master monitoring]#)
kubectl apply -f ground/k8s-1.27/loki-stack-2.6.1

# 설치 확인
kubectl get pods -n loki-stack
```
<div align="center">
<img src="https://github.com/user-attachments/assets/13db64c7-552a-4a70-b09d-7b30dede3ed2">
</div>

5. Grafana 접속
   - 접속 URL : ```http://192.168.56.30:30001```
   - 로그인 :​ (id : admin, pw : admin)
<div align="center">
<img src="https://github.com/user-attachments/assets/3a83ae29-f46f-4b19-a905-352c6166bf1d">
</div>

6. Grafana에서 Loki-Stack 연결
   - Connect data : Home > Connections > Connect data
   - 검색에 [loki] 입력 후 항목 클릭
<div align="center">
<img src="https://github.com/user-attachments/assets/ba62ad77-fa60-4073-8a87-da15636afc4d">
</div>

   - URL에 내용 입력  : ```http://loki-stack.loki-stack:3100```
<div align="center">
<img src="https://github.com/user-attachments/assets/49dd72cb-2686-46f8-be1d-8ffd430d6382">
</div>

   - 하단 Save & Test
<div align="center">
<img src="https://github.com/user-attachments/assets/aed70c6d-89f0-428d-9c6c-43ce33ec0384">
</div>

6. Grafana에 데이터가 안나올 경우
```bash
// System clock synchronized: no 인지 확인
timedatectl 

// yes로 변경
systemctl restart chronyd.service
```

7. 설치를 삭제해야 할 경우, 아래 명령을 이용
   - Prometheus(with Grafana), Loki-stack 삭제 : [k8s-master] Console 접속 후 아래 명령 실행
```bash
[root@k8s-master ~]# cd monitoring

# Prometheus 삭제
kubectl delete --ignore-not-found=true -f ground/k8s-1.27/prometheus-2.44.0/manifests -f ground/k8s-1.27/prometheus-2.44.0/manifests/setup

# Loki-stack 삭제
kubectl delete -f ground/k8s-1.27/loki-stack-2.6.1
```

8. 체험 App배포 - 쿠버네티스 대표 기능
   - App 배포 환경 구성하기
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-1-2-2-1
spec:
  selector:
    matchLabels:
      app: '1.2.2.1'
  replicas: 2
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: '1.2.2.1'
    spec:
      containers:
        - name: app-1-2-2-1
          image: 1pro/app
          imagePullPolicy: Always
          ports:
            - name: http
              containerPort: 8080
          startupProbe:
            httpGet:
              path: "/ready"
              port: http
            failureThreshold: 20
          livenessProbe:
            httpGet:
              path: "/ready"
              port: http
          readinessProbe:
            httpGet:
              path: "/ready"
              port: http
          resources:
            requests:
              memory: "100Mi"
              cpu: "100m"
            limits:
              memory: "200Mi"
              cpu: "200m"
---
apiVersion: v1
kind: Service
metadata:
  name: app-1-2-2-1
spec:
  selector:
    app: '1.2.2.1'
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 31221
  type: NodePort
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-1-2-2-1
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-1-2-2-1
  minReplicas: 2
  maxReplicas: 4
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 40
```
  - App에 지속적으로 트래픽 보내기 (Traffic Routing 테스트)
```bash
[root@k8s-master ~]# while true; do curl http://192.168.56.30:30000/hostname; sleep 2; echo '';  done;
```
   - 트래픽 알고리즘 관련 : 현재 iptalbes 모드로 작동하기 때문에 Random으로 Pod에 트래픽이 분산 (```https://kubernetes.io/ko/docs/reference/networking/virtual-ips/```)

