-----
### Object 그려보며 이해하기
-----
1. 강의 환경 구성하기
   - Master Node에 접속해서 디렉토리 생성
```bash
[root@k8s-master ~]# mkdir -p /root/k8s-local-volume/1231
```

  - Namespace : Object를 Grouping
```yaml
apiVersion: v1
kind: Namespace # 삭제하면, 안에 존재하는 모든 Object도 같이 삭제
metadata:
  name: anotherclass-123
  labels:
    part-of: k8s-anotherclass
    managed-by: dashboard
```

  - Deployment : Pod를 만들고 업그레이드 해주는 역할
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: anotherclass-123 
  name: api-tester-1231 # 💡하나의 Namespace 안에서 같은 종류의 Obejct 이름이 중복되서는 안 됨
  labels:
    part-of: k8s-anotherclass
    component: backend-server
    name: api-tester
    instance: api-tester-1231
    version: 1.0.0
    managed-by: dashboard
spec:
  selector:
    matchLabels:
      part-of: k8s-anotherclass
      component: backend-server
      name: api-tester
      instance: api-tester-1231
  replicas: 2 # Pod 2개 생성
  strategy:
    type: RollingUpdate 
  template: # 이 내용을 바탕으로 Pod 생
    metadata: 
      labels:
        part-of: k8s-anotherclass
        component: backend-server
        name: api-tester
        instance: api-tester-1231
        version: 1.0.0
    spec:
      nodeSelector: # Pod를 생성할 Node 설정
        kubernetes.io/hostname: k8s-master
      containers: 
        - name: api-tester-1231
          image: 1pro/api-tester:v1.0.0
          ports:
          - name: http
            containerPort: 8080
          envFrom: # Application 환경 변수와 관련된 부분 
            - configMapRef: # 그 값을 제공해주는 Configmap
                name: api-tester-1231-properties
          startupProbe: # App이 잘 기동되었는지 확인 (기동이 안 되면 재기동)
            httpGet:
              path: "/startup"
              port: 8080
            periodSeconds: 5
            failureThreshold: 36
          readinessProbe: # App에 트래픽을 연결할 것인지 결정
            httpGet:
              path: "/readiness"
              port: 8080
            periodSeconds: 10
            failureThreshold: 3
          livenessProbe: # App이 비정상이면 재시작할 지 결정
            httpGet:
              path: "/liveness"
              port: 8080
            periodSeconds: 10
            failureThreshold: 3
          resources: # 한 개의 Pod에 CPU, Memory 할당 (미설정 : Pod가 노드의 자원 모두 소진)
            requests:
              memory: "100Mi"
              cpu: "100m"
            limits:
              memory: "200Mi"
              cpu: "200m"
          volumeMounts:
            - name: files
              mountPath: /usr/src/myapp/files/dev # Pod 내부 만들어지는 디렉토리
            - name: secret-datasource
              mountPath: /usr/src/myapp/datasource # Pod 내부 만들어지는 디렉토리
      volumes:
        - name: files # volumeMounts name과 volumes의 name이 서로 매칭되어야 함
          persistentVolumeClaim: # 실제 PVC Object와 연결
            claimName: api-tester-1231-files
        - name: secret-datasources # volumeMounts name과 volumes의 name이 서로 매칭되어야 함
          secret:
            secretName: api-tester-1231-postgresql
```
   - Service
```yaml
apiVersion: v1
kind: Service
metadata: # metadata - namespace, name, labels
  namespace: anotherclass-123
  name: api-tester-1231 # 💡 같은 종류의 Object끼리 이름 중복 불가
  labels:
    part-of: k8s-anotherclass
    component: backend-server
    name: api-tester
    instance: api-tester-1231
    version: 1.0.0
    managed-by: dashboard
spec: # spec - selector, prots, type
  selector: 
    part-of: k8s-anotherclass
    component: backend-server
    name: api-tester
    instance: api-tester-1231
  ports:
    - port: 80
      targetPort: http
      nodePort: 31231
  type: NodePort # Pod에게 트래픽 연결 
```

   - Configmap, Secret
```yaml
apiVersion: v1
kind: ConfigMap # 환경변수 목적적
metadata:
  namespace: anotherclass-123
  name: api-tester-1231-properties
  labels:
    part-of: k8s-anotherclass
    component: backend-server
    name: api-tester
    instance: api-tester-1231
    version: 1.0.0
    managed-by: dashboard
data: # 환경변수값
  spring_profiles_active: "dev"
  application_role: "ALL"
  postgresql_filepath: "/usr/src/myapp/datasource/postgresql-info.yaml"
---
apiVersion: v1
kind: Secret # Pod에 더 중요한 값 제공 목적
metadata:
  namespace: anotherclass-123
  name: api-tester-1231-postgresql
  labels:
    part-of: k8s-anotherclass
    component: backend-server
    name: api-tester
    instance: api-tester-1231
    version: 1.0.0
    managed-by: dashboard
stringData: # 아래 내용을 이용해 Pod 안에 파일 생성
  postgresql-info.yaml: |
    driver-class-name: "org.postgresql.Driver"
    url: "jdbc:postgresql://postgresql:5431"
    username: "dev"
    password: "dev123"
```

  - PVC, PV
```yaml
apiVersion: v1
kind: PersistentVolumeClaim 
metadata:
  namespace: anotherclass-123
  name: api-tester-1231-files
  labels:
    part-of: k8s-anotherclass
    component: backend-server
    name: api-tester
    instance: api-tester-1231
    version: 1.0.0
    managed-by: kubectl
spec:
  resources:
    requests:
      storage: 2G # 저장 공간 설정
  accessModes: # 저장 공간 모드 설정
    - ReadWriteMany # 읽기 / 쓰기
  selector:
    matchLabels:
      part-of: k8s-anotherclass
      component: backend-server
      name: api-tester
      instance: api-tester-1231-files
---
apiVersion: v1
kind: PersistentVolume # 실제 Volume 지정
metadata: # Namespace 없음 (💡 Namespace와 PV은 Cluster-Level Object) / Depolyment, Service는 Namespace Level Obejct / 각 Object들은 자신의 Level에만 생성 가능
  name: api-tester-1231-files
  labels:
    part-of: k8s-anotherclass
    component: backend-server
    name: api-tester
    instance: api-tester-1231-files
    version: 1.0.0
    managed-by: dashboard
spec:
  capacity:
    storage: 2G
  volumeMode: Filesystem 
  accessModes:
    - ReadWriteMany
  local: # Path를 Volume으로 사용
    path: "/root/k8s-local-volume/1231"
  nodeAffinity: # Master 노드 지정
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - {key: kubernetes.io/hostname, operator: In, values: [k8s-master]}
```
   - HPA : 부하에 따라 Pod를 증가, 감소하는 스케일링 역할
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  namespace: anotherclass-123
  name: api-tester-1231-default
  labels:
    part-of: k8s-anotherclass
    component: backend-server
    name: api-tester
    instance: api-tester-1231
    version: 1.0.0
    managed-by: dashboard
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment # 대상 : Deployment 
    name: api-tester-1231
  minReplicas: 2 # 최소 2개 Pod 유지 
  maxReplicas: 4 # 최대 4개까지 생성
  metrics: # Pod의 CPU 사용률이 평균 60%가 늘어나면, Scale-Out 설정
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
  behavior: # 한 번 증가후, 120초 동안 늘어나지 않게 설정 
    scaleUp:
      stabilizationWindowSeconds: 120
```

2. 강의에서 배포한 Object 삭제 (kubectl 명령)
```bash
[root@k8s-master ~]# kubectl delete ns anotherclass-123
[root@k8s-master ~]# kubectl delete pv api-tester-1231-files
```
2.  Object 그려보며 이해하기
    - Object 
<div align="center">
<img src="https://github.com/user-attachments/assets/90394ee4-7129-4a0b-86e9-01ff196cbf1f" />
<img src="https://github.com/user-attachments/assets/d604de67-cc4c-4d3c-824b-2e08b0192eea" />
</div>


