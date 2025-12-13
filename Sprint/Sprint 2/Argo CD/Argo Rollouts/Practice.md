-----
### Argo Rollouts를 이용한 Blue-Green 배포
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/98beffe5-a352-4be3-aaf0-80906680e687" />
</div>

1. Argo Rollouts 설치
   - Jenkins > Dashboard > add-on > deploy-argo > 파라미터와 함께 빌드
```
DEPLOY_TYPE : helm_upgrade
TARGET_ARGO : argocd-rollouts
```
  - values-dev.yaml 파일
```yaml
controller:
  replicas: 1

dashboard:
  enabled: true
  service:
    type: NodePort
    portName: dashboard
    port: 3100
    targetPort: 3100
    nodePort: 30003
```
   - 설치 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/f05e9bed-9f44-432f-a75b-86085f67a11e" />
</div>

2. App 생성 하기 - [+ NEW APP] 클릭
   - [EDIT AS YAML] 선택 후 아래 내용 붙여넣은 후 [CREATE] 하기
      + ArgoRollouts의 기능 확인이 중요한 부분이라 여기서 부터는 Github_Username을 변경 안하고, 아래 내용을 그대로 사용
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: api-tester-2233
spec:
  destination:
    name: ''
    namespace: anotherclass-223
    server: 'https://kubernetes.default.svc'
  source:
    path: 2233/deploy/argo-rollouts
    repoURL: 'https://github.com/k8s-1pro/kubernetes-anotherclass-sprint2.git'
    targetRevision: main
  sources: []
  project: default
```

3. 배포하기 - [SYNC] 클릭 > [SYNCHRONIZE] 클릭
4. 배포 확인
5. 트래픽 보내기  (Active Service (32233) - 1.0.0 App 연결, Preview Service (32243) - 1.0.0 App 연결)
```bash
# Active Service
[root@k8s-master ~]# while true; do curl http://192.168.56.30:32233/version; sleep 2; echo '';  done;
[App Version] : Api Tester v1.0.0
[App Version] : Api Tester v1.0.0

# Preview Service
[root@k8s-master ~]# while true; do curl http://192.168.56.30:32243/version; sleep 2; echo '';  done;
[App Version] : Api Tester v1.0.0
[App Version] : Api Tester v1.0.0
```

6. ArgoCD UI에서 Blue/Green 배포 시작하기
   - 배포 파일
<div align="center">
<img src="https://github.com/user-attachments/assets/80e9d6cc-2c40-4d3b-9d22-ea4c6f8b1351" />
</div>

   - service-active.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-tester-2233-active
  labels:
    part-of: k8s-anotherclass
    ...
spec:
  selector:
    part-of: k8s-anotherclass
    component: backend-server
    name: api-tester
    instance: api-tester-2233
  ports:
    - port: 80
      targetPort: http
      nodePort: 32233
  type: NodePort
▶ rollout.yaml 설정 내용

apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: api-tester-2233
  labels:
    part-of: k8s-anotherclass
     ...
spec:
  replicas: 2
  strategy:
    blueGreen:
      activeService: api-tester-2233-active
      previewService: api-tester-2233-preview
      autoPromotionEnabled: false
  ...
  template:
    metadata:
      labels:
        part-of: k8s-anotherclass
        component: backend-server
        name: api-tester
        instance: api-tester-2234
        version: 1.0.0
```
  - strategy.blueGreen
    + previewService : 업그레이드 중간에 새로 만들어지는 RepliaSet의 Pod에 먼저 연결되는 서비스 
    + activeService : 주 서비스 (업그레이 시 트래픽이 Blue에서 Green으로 변경됨)
    + autoPromotionEnabled : 새 RepliaSet이 활성화 되면 바로 트래픽을 전환 = 자동 업그레이드 (default : true)
    + ```https://argo-rollouts.readthedocs.io/en/stable/features/bluegreen/```
   
   - HPA 생성시 예제
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: api-tester-2233
spec:
  maxReplicas: 4
  minReplicas: 2
  scaleTargetRef:
    apiVersion: argoproj.io/v1alpha1
    kind: Rollout
    name: api-tester-2233
  targetCPUUtilizationPercentage: 80
```
​  
  - Git에서 image의 tag 변경 (1.0.0 -> 2.0.0로 변경)
```yaml
    spec:
      containers:
        - envFrom:
            - configMapRef:
                name: api-tester-2233-properties
          image: '1pro/api-tester:1.0.0'  // -> '1pro/api-tester:2.0.0' 로 변경
```
​  - ArgoCD에서 Applications > api-tester-2233 > [SYNC] 클릭
  - 트래픽 확인 (Active Service (32233) - 1.0.0 App 연결, Preview Service (32243) - 2.0.0 App 연결)
```bash
# Active Service
[App Version] : Api Tester v1.0.0
[App Version] : Api Tester v1.0.0

# Preview Service
[App Version] : Api Tester v2.0.0
[App Version] : Api Tester v2.0.0
```
  - Promote 진행
    + ArgoCD > Applications > api-tester-2233 > Rollout > [...]클릭 > promote-Full
<div align="center">
<img src="https://github.com/user-attachments/assets/5bc025cf-f093-4ebe-809d-04e703702e53" />
</div>

  - Argo Rollouts Dashboard
```bash
http://192.168.56.30:30003/rollouts/anotherclass-223
```

<div align="center">
<img src="https://github.com/user-attachments/assets/53e8f5fd-1ad8-45a8-85c2-0697ee0007ef" />
</div>

  - 트래픽 확인 (Preview Service (32233) - 2.0.0 App 연결, Preview Service (32243) - 2.0.0 App 연결)
```bash
# Active Service
[App Version] : Api Tester v2.0.0
[App Version] : Api Tester v2.0.0

# Preview Service
[App Version] : Api Tester v2.0.0
[App Version] : Api Tester v2.0.0
```
   - Rollout CLI로 조회 해보기 : Argo Rollouts CLI 설치 (v1.6.4) 
     + Windows
```bash
// root 계정
curl -LO https://github.com/argoproj/argo-rollouts/releases/download/v1.6.4/kubectl-argo-rollouts-linux-amd64
chmod +x ./kubectl-argo-rollouts-linux-amd64
mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
```
  - 설치 확인
```bash
[root@k8s-master ~]# kubectl argo rollouts version
kubectl-argo-rollouts: v1.6.4+a312af9
  BuildDate: 2023-12-11T18:31:15Z
  GitCommit: a312af9f632b985ec13f64918b918c5dcd02a15e
  GitTreeState: clean
  GoVersion: go1.20.12
  Compiler: gc
  Platform: linux/amd64
```
  - 조회하기
```bash
// Rollout 조회 하기
kubectl argo rollouts get rollout api-tester-2233 -n anotherclass-223 -w
https://argo-rollouts.readthedocs.io/en/stable/generated/kubectl-argo-rollouts/kubectl-argo-rollouts/
```

7. 리소스 정리하기
<div align="center">
<img src="https://github.com/user-attachments/assets/bf5b00d1-4c5c-433d-9057-915822c63706" />
</div>

-----
### Argo Rollouts를 이용한 Carary 배포
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/afc18c2f-416d-476b-9cde-758fb94a9fb6" />
</div>

1. Master에서 Kubectl로 Rollouts 배포하기
```bash
[root@k8s-master ~]#
kubectl apply -f https://raw.githubusercontent.com/k8s-1pro/kubernetes-anotherclass-sprint2/main/2234/deploy/argo-rollouts/rollout.yaml -n anotherclass-223
kubectl apply -f https://raw.githubusercontent.com/k8s-1pro/kubernetes-anotherclass-sprint2/main/2234/deploy/argo-rollouts/configmap.yaml -n anotherclass-223
kubectl apply -f https://raw.githubusercontent.com/k8s-1pro/kubernetes-anotherclass-sprint2/main/2234/deploy/argo-rollouts/secret.yaml -n anotherclass-223
kubectl apply -f https://raw.githubusercontent.com/k8s-1pro/kubernetes-anotherclass-sprint2/main/2234/deploy/argo-rollouts/service.yaml -n anotherclass-223
```

​2. 배포 모니터링
```bash
// Rollout 조회 하기
kubectl argo rollouts get rollout api-tester-2234 -n anotherclass-223 -w
```

3. 트래픽 보내기 (1.0.0 App 연결)
```bash
# Service
[root@k8s-master ~]# while true; do curl http://192.168.56.30:32234/version; sleep 2; echo '';  done;
[App Version] : Api Tester v1.0.0
[App Version] : Api Tester v1.0.0
```

4. rollout.yaml 설정 내용
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: api-tester-2234
  labels:
    part-of: k8s-anotherclass
    ...
spec:
  replicas: 2
  strategy:
    canary:
      steps:                        
        - setWeight: 33         
        - pause: {}                
        - setWeight: 66         
        - pause: { duration: 2m }  
  selector:
    matchLabels:
      ...  // (이하 Deployment Spec과 상동)
```

   - strategy.canary (ex) pause)
     + pause: { duration: 10 }  # 10 seconds
     + pause: { duration: 10s } # 10 seconds
     + pause: { duration: 10m } # 10 minutes
     + pause: { duration: 10h } # 10 hours
     + ```https://argo-rollouts.readthedocs.io/en/stable/features/canary/```

5. Argo Rollouts CLI로 Canary 배포 시작하기 (1.0.0 -> 2.0.0로 변경)
```bash
// kubectl argo rollouts edit <ROLLOUT_NAME> -n <NAMESPACE_NAME>
kubectl argo rollouts edit api-tester-2234 -n anotherclass-223

// kubectl argo rollouts set image <ROLLOUT_NAME> <CONTAINER_NAME>=<IMAGE>:<TAG> -n <NAMESPACE_NAME>
kubectl argo rollouts set image api-tester-2234 api-tester-2234=1pro/api-tester:2.0.0 -n anotherclass-223
```
​
6. Argo Rollout Dashboard에서 [Detail 화면 보기]에서 [Promote] 클릭
```
http://192.168.56.30:30003/rollouts/rollout/anotherclass-223/api-tester-2234
```
<div align="center">
<img src="https://github.com/user-attachments/assets/150ef8e7-02fb-4ef0-9495-8cb39d1485ad" />
</div>

   - Step1. Set Weight: 33% : Stable​ 2 pod, Canary 1 pod 상태로 만듬
   - Step2. Pause : Promote 클릭 때까지 멈춤
   - Step3. Set Weight: 66% : Stable 1 pod, Canary 2 pod 상태로 만듬
   - Step4. Pause (2m) : 2분 동안 멈춤
   - Step5. : Canary 2 pod 상태로 만듬
  
   - CLI
```bash
// kubectl argo rollouts promote <ROLLOUT_NAME> -n <NAMESPACE_NAME>
kubectl argo rollouts promote api-tester-2234 -n anotherclass-223
```

7. 트래픽 확인 (1.0.0 App 연결, 2.0.0 App 연결)
```bash
# Service
[root@k8s-master ~]# while true; do curl http://192.168.56.30:32234/version; sleep 2; echo '';  done;
[App Version] : Api Tester v1.0.0
[App Version] : Api Tester v1.0.0
[App Version] : Api Tester v2.0.0   // canary
```
​
8. 트래픽 확인 (2.0.0 App 연결)
```bash
# Service
[root@k8s-master ~]# while true; do curl http://192.168.56.30:32234/version; sleep 2; echo '';  done;
[App Version] : Api Tester v2.0.0
[App Version] : Api Tester v2.0.0
```

9. 리소스 정리하기
```bash
[root@k8s-master ~]#
kubectl delete -f https://raw.githubusercontent.com/k8s-1pro/kubernetes-anotherclass-sprint2/main/2234/deploy/argo-rollouts/rollout.yaml -n anotherclass-223
kubectl delete -f https://raw.githubusercontent.com/k8s-1pro/kubernetes-anotherclass-sprint2/main/2234/deploy/argo-rollouts/configmap.yaml -n anotherclass-223
kubectl delete -f https://raw.githubusercontent.com/k8s-1pro/kubernetes-anotherclass-sprint2/main/2234/deploy/argo-rollouts/secret.yaml -n anotherclass-223
kubectl delete -f https://raw.githubusercontent.com/k8s-1pro/kubernetes-anotherclass-sprint2/main/2234/deploy/argo-rollouts/service.yaml -n anotherclass-223
```
