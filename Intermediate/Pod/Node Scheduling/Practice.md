-----
### 실습 - Node Scheduling
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/f6b5e97b-62ca-4efa-991b-038a32d89fcc">
</div>

1. Node Affinity
    - NodeSelector 보다 좀더 다양한 방법으로 Node에 스케줄링을 할 수 있는 기능들을 제공
    - matchExpression : 해당 규칙에 부합하는 Node에 배치, 여러 Node가 있을 경우 자원이 많은 곳에 배치됨
    - required : 해당 옵션 지정시 절대적으로 matchExpression에 부합하는 Node에만 배치됨
    - preferred : 해당 옵션 지정시 matchExpression에 부합하는 Node가 없으면, 자원이 많은 Node에 배치됨
    - preferred.weight : 두 개 이상의 preferred 옵션을 설정시, 좀더 선호하는 옵션으로 가중치를 부여함
    - Node Labeling 
```bash
kubectl label nodes k8s-worker1 kr=az-1
kubectl label nodes k8s-worker2 us=az-1
```
   - MatchExpressions
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod-match-expressions1
spec:
 affinity:
  nodeAffinity:
   requiredDuringSchedulingIgnoredDuringExecution:   
    nodeSelectorTerms:
    - matchExpressions:
      -  {key: kr, operator: Exists} # Key가 kr인 Label Matching
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```
  - Required : Node에 key로 ch가 없기 때문에 Pending 상태
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod-required
spec:
 affinity:
  nodeAffinity:
   requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
    - matchExpressions:
      - {key: ch, operator: Exists} # 해당 Key가 업으므로 Pending
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```
   - Preferred : Node에 key로 ch가 없지만 prefered 옵션이기 때문에 자원이 많은 Node에 할당
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod-preferred
spec:
 affinity:
  nodeAffinity:
   preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 1 # 가중치
      preference:
       matchExpressions:
       - {key: ch, operator: Exists}
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```

   - 실습 후 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 or Master Node에서 아래 명령 실행)
```bash
kubectl delete pod pod-match-expressions1 pod-required pod-preferred
kubectl label nodes k8s-worker1 kr-
kubectl label nodes k8s-worker2 us-
```

2. Pod Affinity / Anti-Affinity
    - Node가 아닌 Pod의 Label에 따라 배치 방법이 결정됨
    - Pod Affinity : Pod를 서로 같은 노드에 배치
    - Pod Anti-Affinity : Pod를 서로 다른 노드에 배치
    - topologyKey : Pod에 따라 배치가 되더라도, Node의 범위를 key를 통해 지정가능
    - Node Labeling
```bash
kubectl label nodes k8s-worker1 a-team=1
kubectl label nodes k8s-worker2 a-team=2
```
   - Web1 Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: web1
 labels:
  type: web1
spec:
 nodeSelector:
  a-team: '1'
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```
   - Web1 Affinity Pod : web1 Pod가 배치된 노드에 생성
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: server1
spec:
 affinity:
  podAffinity:
   requiredDuringSchedulingIgnoredDuringExecution:   
   - topologyKey: a-team
     labelSelector:
      matchExpressions:
      -  {key: type, operator: In, values: [web1]}
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```
   - Web2 Affinity Pod : web2 Pod가 없기 때문에 노드에 할당되지 않고 Pending 상태
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: server2
spec:
 affinity:
  podAffinity:
   requiredDuringSchedulingIgnoredDuringExecution:   
   - topologyKey: a-team
     labelSelector:
      matchExpressions:
      -  {key: type, operator: In, values: [web2]}
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```
   - Web2 Pod : web2 Pod를 만들어 줬기 때문에 server2 Pod는 노드에 할당
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web2
  labels:
     type: web2
spec:
  nodeSelector:
    a-team: '2'
  containers:
  - name: container
    image: kubetm/app
  terminationGracePeriodSeconds: 0
```
   - Master Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: master
  labels:
     type: master
spec:
  nodeSelector:
    a-team: '1'
  containers:
  - name: container
    image: kubetm/app
  terminationGracePeriodSeconds: 0
```
   - Master Anti-Affinity Pod 
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: slave
spec:
 affinity:
  podAntiAffinity:
   requiredDuringSchedulingIgnoredDuringExecution:   
   - topologyKey: a-team
     labelSelector:
      matchExpressions:
      -  {key: type, operator: In, values: [master]}
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```
  - 실습 후 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 or Master Node에서 아래 명령 실행)
```bash
kubectl delete pod web1 server1 web2 server2 master slave
kubectl label nodes k8s-worker1 a-team-
kubectl label nodes k8s-worker2 a-team-
```

3. Taint / Toleration
    - Taint : Pod가 특정 Node에 배치되는 걸 제한하기 위해 사용
    - Toleration : Taint가 있는 Node에 Pod를 배치할 때 사용
    - 할당 시킬 Node 지정은 Taint/Toleration과 관계 없이 NodeSelector를 사용해야 함
    - effect.NoSchedule : 해당 Taint가 있는 노드에 Pod가 스케줄링 될 수 없음 (Master에 기본적으로 설정되어 있음 : User가 만든 Pod가 MasterNode에 배치되지 않게 하기 위함)    
    - effect.PreferNoSchedule : 해당 Taint가 있는 노드에 스케줄링 될 수 없으나 정 배치될 Node가 없으면 할당됨    
    - effect.NoExecute : Taint 부여시 해당 Node위에 있는 모든 Pod는 삭제됨 (Node 장애시 쿠버네티스가 자동으로 해당 Node위에 Taint를 생성시킴 : Pod가 다른 노드위에 만들어지게 하기 위함)    
    - Toleration.tolerationSeconds : 설정된 시간이 지난 후에 Pod를 삭제 시킴 (Node의 일시적인 장애 일수도 있기 때문에 일정 시간동안 지연시킴)
    - Worekr Node1에 Labeling 적용
```bash
kubectl label nodes k8s-worker1 gpu=no1
```
   - Worekr Node1에 Taint 적용
```bash
kubectl taint nodes k8s-worker1 hw=gpu:NoSchedule
```
  - Pod with Toleration
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod-with-toleration
spec:
 nodeSelector:
  gpu: no1
 tolerations:
 - effect: NoSchedule
   key: hw
   operator: Equal
   value: gpu
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```
   - Pod without Toleration : toleration이 없는 Pod는 Worekr Node1에 할당되지 않음
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod-without-toleration
spec:
 nodeSelector:
  gpu: no1
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```
   - Pod1 with NoExecute : Node에 Taint(NoExecute)가 부여되면 해당 Pod는 30초 후에 삭제됨
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod1-with-no-execute
spec:
 tolerations:
 - effect: NoExecute
   key: out-of-disk
   operator: Exists
   tolerationSeconds: 30
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```
   - Pod2 with NoExecute : Node에 Taint(NoExecute)가 부여되도 해당 Pod는 삭제되지 않음
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod2-without-no-execute
spec:
 tolerations:
 - effect: NoExecute
   key: out-of-disk
   operator: Exists
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```
   - Worker Node2에 Taint 적용
```bash
kubectl taint nodes k8s-worker1 hw=gpu:NoSchedule-
kubectl taint nodes k8s-worker2 out-of-disk=True:NoExecute
```
   -  실습 후 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 or Master Node에서 아래 명령 실행)
```bash
kubectl delete pod pod-with-toleration pod-without-toleration pod1-with-no-execute pod2-without-no-execute
kubectl taint nodes k8s-worker2 out-of-disk=True:NoExecute-
```

4. NodeAffinity MatchExpressions Operator
<div align="center">
<img src="https://github.com/user-attachments/assets/cf3eaf88-e9f1-4417-99ec-efae34e73b12">
</div>

