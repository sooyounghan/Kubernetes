-----
### HPA (Auto Scaling)
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/ad8ea3ce-90ba-4b84-a890-3fb519eb3d97" />
</div>

1. scaleTargetRef : Deployment를 지정하는 내용
2. minReplicas, maxReplicas : 부하 시 범위 내에서 ReplicaSet 변경
3. metrics
   - target.averageUtilization: 60 (CPU가 평균 60% 기준으로 Scaling)
     + Pod의 resource와 관련, reqeust의 CPU는 100m, memory는 100MB로 설정 (HPA에서 100%를 계산하기 위한 기준 값)
     + conatiner 내부에는 App이 기동 중이며, 이 App에서 실제 사용 중인 CPU가 계산 : 예) 70m CPU라면, Pod 2개 중 하나만 70%이므로 평균으로 하면 35%이므로 변화가 없음, 150% 정도 상승하면 평균 60%을 넘으면서 스케일링 시작
     + 계산 공식 : 현재 Pod * (평균 CPU / HPA CPU) - 변경될 Pod의 수
       * 2.51가 나오면, 올림 처리가 되어 새 Pod 하나 생성
     + resouces.limits 속성 : CPU는 최대 200m을 넘지 않도록 함 (즉, 한 Pod에서 최대 200%까지 넘지 않도록 하는 것임)
       * limits.cpu의 속성 값이 reqeust.cpu 값과 동일하다면, 한 Pod에는 100% 이상 CPU가 올라갈 수 없으므로 HPA의 임게치를 낮추지 않는 이상 Pod 하나만 부하가 증가한다고 스케일링이 일어나지 않음
       * 즉, limits가 HPA에 의해 평균 임계치를 결정하는데 큰 영향을 줌
     + memory의 경우 CPU와 같이 사용할 때 상승, 미사용 시 내려가는 자원이 아닌 App을 기동시킬 때 내부적으로 메모리 최대 사용량 설정(-Xmx 50M)하는데, 처음 40MB 정도 사용하다가 점점 늘어나면서 50MB가 넘어가게 되면, App 내부 Garbage Collection이 실행되면서 쓰지 않는 메모리를 정리하고 메모리 수치를 낮춤
       * 즉, 사용 상황에 따라 변하는 값이지 부하에 따른 증감 대상이 아니므로, HPA에서 제공하는 옵션이지만 거의 쓰지 않음

4. 이상적인 스케일링 동작
   - 처음 Pod 2개가 서비스 되고 있는 상태에서 부하가 꾸준하게 증가
   - 평균 60%가 넘으면, Scale-Out이 되고, Pod 하나가 새로 생성되며, 기동이 완료되면 평균 부하가 떨어짐
   - 트래픽이 감소되면 서서히 부하가 낮아지기 시작하는데, 일정 구간 밑으로 떨어지면 Scale-In이 발생하면서 Pod가 하나 삭제되어 평균 CPU가 올라감

5. 현실적인 스케일링 동작
   - 부하가 증가하면 CPU가 급격하게 증가하면서 Pod는 모두 죽음
   - CPU가 100%을 넘으므로 Kubernetes는 Scale-Out을 하는데, 기존 죽어 있는 Pod 2개는 새로 기동되어 있는 상태지만, 그 시간만큼 서비스 중단이 발생
   - Pod가 모두 기동되면서 서비스가 안정화되지만, 부하가 잦아지면 마찬가지로 Scale-In이 자주 일어나고, CPU가 올라감
   - 즉, 자동화 스케일링 기능은 보조적인 역할로 사용해야 하며, 부하는 서비스마다 Peak 시간을 분석해 미리 자원을 증가 또는 대기열 아키테쳐를 구성하는 것이 좋ㅇ

6. behavior
   - 잦은 스케일링을 방지하기 위한 목적 (부하가 잠깐 올라갔다가 감소하는 현상 발생 가능(자바 애플리케이션의 경우 API 하나의 복잡한 로직이 있다면 CPU가 급격하게 증가하기 쉬운데, Scale-Out이 되면, 새 Pod가 만들어지고, 죽는 과정에서 CPU가 비효율적으로 사용됨)
   - scaleUp.stabilizationWindowSeconds(안정화 윈도우) 설정을 하면, 2분 정도 CPU가 60% 이상을 유지헤야 Scale-Out이 발생
     + Scale Up / Down : 한 서버의 자원을 높이거나 낮출 때 사용
     + Sclae In / Out : 서버 개수를 늘리거나 감소시킬 때 사용
  - scaleDown.stabilizationWindowSeconds(안정화 윈도우) 설정 : Pod 수가 줄어들 때도 안정화 윈도우 기능 사용 가능 (여기서는 부하가 언제 올지 모르므로 10분을 기다리도록 설정)
  - policies
    + types: Pods / value : 1 / periodSeconds : 60 (삭제할 파드를 1분에 1개씩 제거)
    + 즉, 삭제할 Pod가 5개 일지라도, 한 번에 삭제하지 않고, 1분에 1개씩 삭제해도록 설정

7. 동작 방식
<div align="center">
<img src="https://github.com/user-attachments/assets/bbca4f76-c73c-4651-86fc-265cb73a07d4" />
</div>

  - 부하 발생
```bash
http://192.168.56.30:31231/cpu-load?min=3
// 3분 동안 부하 발생
```
   - Pod에 부하가 심할 경우 livenessProbe에 의해 파드가 restart 될 수 있음
```bash
// 개인 PC 사양(core 수)에 따라 부하가 너무 오르거나/오르지 않을 경우 queryparam으로 수치 조정
http://192.168.56.30:31231/cpu-load?min=3&thread=5

// 3분 동안 5개의 쓰레드로 80% 부하 발생 
// default : min=2, thread=10
```
​
   - 부하 확인
```bash
// 실시간 업데이트는 명령어로 확인하는 게 빨라요
kubectl top -n anotherclass-123 pods
kubectl get hpa -n anotherclass-123
```

```bash
// Grafana는 Prometheus를 거쳐 오기 때문에 좀 늦습니다
Grafana > Home > Dashboards > [Default] Kubernetes / Compute Resources / Pod
```

  - 예상치 않은 상황 발생시 상태 원복 방법
```bash
// 1. hpa 삭제
kubectl delete -n anotherclass-123 hpa api-tester-1231-default
```
```bash
// 2. deployment replicas 2로 변경
kubectl scale -n anotherclass-123 deployment api-tester-1231 --replicas=2
```
```bash
// 3. hpa 다시 생성
kubectl apply -f - <<EOF
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
    kind: Deployment
    name: api-tester-1231
  minReplicas: 2
  maxReplicas: 4
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 120
EOF
```

  - [behavior] 미사용으로 적용
```bash
kubectl edit -n anotherclass-123 hpa api-tester-1231-default
```
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  namespace: anotherclass-123
  name: api-tester-1231-default
spec:
  behavior:  # 삭제
    scaleUp:   # 삭제
      stabilizationWindowSeconds: 120   # 삭제
```

  - 부하 발생 API 
```bash
http://192.168.56.30:31231/cpu-load 
// 2분 동안 10개의 쓰레드로 80% 부하 발생
// default : min=2, thread=10
```

  - 부하 확인 (kubectl)
```bash
// 실시간 업데이트는 명령어로 확인하는 게 빠름
kubectl top -n anotherclass-123 pods
kubectl get hpa -n anotherclass-123
```

  - 부하 확인 (grafana)
```bash
// 성능 (Prometheus를 거쳐 오기 때문에 좀 표시가 늦음)
Grafana > Home > Dashboards > [Default] Kubernetes / Compute Resources / Pod
```
```bash
// Replica 보기 (Grafana Dashbaord에서 [Kubernetes / Horizontal Pod Autoscaler] 다운로드
Grafana > Home > Dashboards > New > Import 클릭
- Import via grafana.com 항목에 17125 입력 후 [Load] 클릭
```
