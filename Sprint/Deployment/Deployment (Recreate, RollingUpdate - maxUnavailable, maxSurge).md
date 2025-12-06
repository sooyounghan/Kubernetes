-----
### Deployment (Recreate, RollingUpdate - maxUnavailable, maxSurge)
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/33efdc4b-dc1b-4f6e-9329-4b30da7c4aeb" />
</div>

1. App에 새로운 버전이 나와 업데이트를 해야 될 때, 기존 Pod는 삭제하고 새 Pod를 배포해야 되는데, Kubernetes는 Deployment라는 Object에서 strategy라는 속성으로 기능 지원
   - 두 가지 방식 : ReCreate, RollingUpdate
   - Object Update : template 예하 내용이 하나라도 변경이 되면 업데이트가 진행 : 이 내용들은 모두 Pod로 설정되는 내용이므로 Kubernetes는 그 내용으로 새로운 Pod 생성
     + 새로운 이름의 ReplicaSet을 생성 : 2이므로 Pod를 2개 만들면서, 기존의 ReplicaSet을 0으로 변경하고, 이전 Pod는 삭제 (기존 ReplicaSet은 그대로 존재하는데, 이는 새 버전 문제가 생길 때 Rollback 용도)

2. ReCreate
   - Pod가 2개 있는 상태에서 Kubernetes는 바로 기존 Pod를 삭제시키는 동시에 새 Pod 2개 생성 : 처리되는 속도에 따라 기동되는 시간 차이가 발생
   - 첫 번째 Pod가 기동되는 시간만큼 서비스 중단이 업데이트 과정에서 생긴다는 특징 존재

3. RollingUpdaate
   - 새 버전의 Pod를 하나 생성하고, 기동이 완료되자마자 기존 Pod는 삭제
   - 업데이트 중 서비스 중단이 없다는 특징이 존재
   - 단점으로는 업데이트 중 자원 사용량이 150% 증가
   - 또한, 업데이트 중 두 버전이 동시에 호출된다는 특징이 존재
   - Blue / Green : 업데이트 동안 두 버전이 동시에 호출되지 않도록 설정 가능
     + 하지만 자원 사용량이 200%로 증가
     + 별도 배포 솔루션에서 설치해야 제공 가능

   - maxUnavailable
     + 디폴트 값이 2%로 들어가서 RollingUpdate를 할 때 적용
     + 즉, 업데이트 동안 최대 몇 개의 Pod를 서비스 상태로 유지할지 결정
     + 예) 100%로 설정
       * Pod를 모두 삭제해 업데이트 동안 서비스 불가 상태 Pod가 100%에 진입
       * 서비스 중단을 발생시킨다는 의미

   - maxSurge
     + 새 Pod를 최대 몇 개까지 동시에 생성할지 결정
     + 예) 100%로 설정
       * 하나씩 Pod를 생성하는 것이 아닌 ReplicaSet의 수치로 Pod 생성
       * 즉, ReCreate와 같은 효과

   - maxUnavailable : 0%, maxSurge : 100%
     + Pod 2개가 동시에 생성되며, 업데이트 도중 서비스가 되는 Pod를 2개 유지하는 것
     + 기존에 서비스하던 ReplicaSet 수치가 2이므로, 업데이트 동안 살아있는 Pod가 꾸준히 2개가 되도록 설정
     + 즉, 2개의 Pod가 서비스 상태로 유지되며, 새 Pod가 기동되면 기존 Pod가 죽어서 업데이트가 끝날 때까지 2개를 유지
     + 즉, maxAvailable을 0으로 준다는 것은 서비스 중 부하를 계속 유지하기 위해, Pod 개수를 감소시키지 않는다는 것

   - maxUnavailable : 25%, maxSurge : 25% (기본값)
     + 배포가 시작되자마자 2개가 생성되고, 1개가 삭제되면서 이 순간 5개의 Pod로 서비스를 하던 것이 4개가 됨
     + Pod 수가 20개라면, 차이는 더 커짐
     + 자원을 200% 사용하지만, 업데이트 시간 단축
     + Pod 간 기동 시간이 비슷하면 Blue/Green과 비슷한 효과

4. 동작 확인
<img width="1198" height="606" alt="image" src="https://github.com/user-attachments/assets/386b1bd5-64f9-4d0a-92b9-d9006959d460" />

  - RollingUpdate 하기
```bash
// 1) HPA minReplica 2로 바꾸기 (이전 강의에서 minReplicas를 1로 바꿔놨었음)
kubectl patch -n anotherclass-123 hpa api-tester-1231-default -p '{"spec":{"minReplicas":2}}'
```

```bash
// 1) 그외 Deployment scale 명령
kubectl scale -n anotherclass-123 deployment api-tester-1231 --replicas=2
```

```bash
// 1) edit로 모드로 직접 수정
kubectl edit -n anotherclass-123 deployment api-tester-1231

// 2) 지속적으로 Version호출 하기 (업데이트 동안 리턴값 관찰)
while true; do curl http://192.168.56.30:31231/version; sleep 2; echo ''; done; 

// 3) 별도의 원격 콘솔창을 열어서 업데이트 실행 
kubectl set image -n anotherclass-123 deployment/api-tester-1231 api-tester-1231=1pro/api-tester:v2.0.0
kubectl set image -n anotherclass-123 deployment/api-tester-1231
```

```bash
// update 실행 포맷 
// kubectl set image -n <namespace> deployment/<deployment-name> <container-name>=<image-name>:<tag>
```

  - RollingUpdate (maxUnavailable: 0%, maxSurge: 100%) 하기
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: anotherclass-123
  name: api-tester-1231
spec:
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25% -> 0%  # 수정
      maxSurge: 25% -> 100%      # 수정
```

```bash
kubectl set image -n anotherclass-123 deployment/api-tester-1231 api-tester-1231=1pro/api-tester:v1.0.0
```
​
  - Recreate 하기
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: anotherclass-123
  name: api-tester-1231
spec:
  replicas: 2
  strategy:
    type: RollingUpdate -> Recreate   # 수정
    rollingUpdate:        # 삭제
      maxUnavailable: 0%  # 삭제
      maxSurge: 100%      # 삭제
```

```bash
kubectl set image -n anotherclass-123 deployment/api-tester-1231 api-tester-1231=1pro/api-tester:v2.0.0
```
  - Rollback
```bash
// 이전 버전으로 롤백
kubectl rollout undo -n anotherclass-123 deployment/api-tester-1231
```
