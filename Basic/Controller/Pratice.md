-----
### Pratice (ReplicaSet - Template, Replicas, Selector)
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/665f0838-7eb3-438f-a00a-7a8035faa9a0">
</div>

1. Template / Replicas
   - Template : Controller에서 Pod의 스펙을 입력하는 설정
   - Replicas : Controller에 연결된 Pod의 개수를 조절하는 설정
   - Pod를 먼저 생성 후 ReplicaSet.Template의 name과 selector을 일치시키면 해당 Pod와 연결됨
   - 하지만 보통 ReplicaSet만 생성하고, 그럴 경우 ```<ReplicaSet-Name>-<Random>```의 이름으로 Pod 생성됨
   - Pod 삭제시 ReplicaSet이 Template의 내용을 토대로 Replicas 만큼 Pod 수를 유지함
   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    type: web # label
spec:
  containers:
  - name: container
    image: kubetm/app:v1
  terminationGracePeriodSeconds: 0 # Pod를 삭제시키면 기본적으로 30초 후 삭제가 되도록 설정되어 있는 시간을 변경
```
   - ReplicaSet
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica1
spec:
  replicas: 1 # Pod가 1개있으므로 replicaSet 1로 지
  selector:
    matchLabels: 
      type: web # key, value를 pod와 일치하게 지정
  template: # Pod의 내용과 동
    metadata:
      name: pod1
      labels:
        type: web
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
      terminationGracePeriodSeconds: 0
```
   - 상단에 리소스 스케일로 스케일 지정 가능
<div align="center">
<img src="https://github.com/user-attachments/assets/f3f8e463-fe21-4d06-9b81-f6bcd56fd42d">
</div>

   - 설정된 Scale 값을 맞추기 위해 임의의 Pod 자동 생성 (ReplicaSet 이름 + 임의의 문자열) : 템플릿에 Pod의 내용을 읽을 떄 name을 설정하면, 해당 이름으로 지정 가능
<div align="center">
<img src="https://github.com/user-attachments/assets/fc610588-893c-48ca-a9c7-a0e30c5beb23">
</div>

   - Pod를 삭제하면, Controller는 이 Replica 개수만큼 유지를 위해 Pod를 생성
<div align="center">
<img src="https://github.com/user-attachments/assets/555ef559-cbb5-431c-bc36-cd34bbb68e60">
</div>

   - 수동으로 Pod 버전 업데이트 (ReplicaSet에서 변경하고, Pod 삭제 후 재생성되면 수동 업데이트) 
<div align="center">
<img src="https://github.com/user-attachments/assets/8437b40a-e223-43e2-8a51-efb7d112a8ce">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/afb6c45b-fdfe-43b6-a2e2-621b87acba2f">
</div>

   - ReplicaSet를 삭제하게 되면 Pod 모두 삭제

   - Replicas 변경 (Dashboard에서 해당 ReplicaSet의 스케일 변경 또는 아래 kubectl 명령)
```bash
kubectl scale --replicas=2 replicaset replica1
```
   - Pod만 남기고 Controller(ReplicaSet)만 삭제하는 방법
```bash
kubectl delete replicaset replica1 --cascade=orphan
```
   - ReplicationController로 했지만, ReplicationController은 현재 사용 안하기 때문에 ReplicaSet로 사용

2. Selector
   - selector.matchLabels : key:value를 이용한 기본적인 매칭 방법
   - selector.matchExpressions : 조건식을 이용해서 좀더 세밀하게 매칭 방법을 컨트롤 가능하며, 일반적으로 Pod를 Node에 배치시킬 때 주로 사용
   - ReplicaSet
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica2
spec:
  replicas: 1
  selector:
    matchLabels: # 자주 사용 (이 내용은 템플릿에 있는 라벨의 내용에 포함되어야 함)
      type: web
      ver: v1
    matchExpressions: # 사용 빈도가 적음 : 사전에 어떤 Object가 생성되었고, 이 Object들이 여러 Label에 붙어있을 때, 원하는 Object를 세밀하게 선택하기 위해 사용
    - {key: type, operator: In, values: [web]}
    - {key: ver, operator: Exists}
  template:
    metadata:
      labels: # matchLabels에 포함
        type: web
        ver: v1
        location: dev
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
      terminationGracePeriodSeconds: 0
```
   - 다음의 경우에는 에러가 발생
```yaml
  selector:
    matchLabels:
      type: web
      ver: v1
  template:
    metadata:
      labels:
        type: web
        ver: v1
        ver: v2
```
​
   - 실습 후 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 또는 Master Node에서 아래 명령 실행)
```bash
kubectl delete rs replica1 replica2
```

3. MatchExpressions
<div align="center">
<img src="https://github.com/user-attachments/assets/af28c012-3590-4e65-9b63-9351bc84db42">
</div>
