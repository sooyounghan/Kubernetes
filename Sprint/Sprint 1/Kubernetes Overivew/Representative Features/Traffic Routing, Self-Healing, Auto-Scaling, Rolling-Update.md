-----
### Kubernetes 대표 기능 - Traffic Routing, Self-Healing, Auto-Scaling, Rolling-Update
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/50c120d0-48a5-433f-aae7-1e97f10a681a">
</div>

1. App에 지속적으로 트래픽 보내기 (Traffic Routing 테스트)
```bash
[root@k8s-master ~]# while true; do curl http://192.168.56.30:31221/hostname; sleep 2; echo '';  done;
```
   - 트래픽 알고리즘 관련 : 현재 iptalbes 모드로 작동하기 때문에 Random으로 Pod에 트래픽이 분산 (```https://kubernetes.io/ko/docs/reference/networking/virtual-ips/```)

2. App에 Memory Leak 나게 하기 (Self-Healing 테스트)
```bash
[root@k8s-master ~]# curl 192.168.56.30:31221/memory-leak
```

3. App에 부하주기 (AutoScaling 테스트)
```bash
[root@k8s-master ~]# curl 192.168.56.30:31221/cpu-load
```
   - 사용자 환경에 따라 API를 받은 Pod의 CPU가 너무 증가해서 Restart가 될 수 있음
   - 이때 스케일링이 되지 않았다면 다시 시도
​
4. App 이미지 업데이트 (RollingUpdate 테스트) (Namespace: default > 디플로이먼트 > ... > 편집)
<div align="center">
<img src="https://github.com/user-attachments/assets/eb7ff123-be70-4bb4-ac32-1f5ecd0a32c8">
</div>

```bash
spec:
      containers:
        - name: app-1-2-2-1
          image: 1pro/app-update  # 수정
```

   - kubectl 명령으로 할 경우
```bash
[root@k8s-master ~]# kubectl set image -n default deployment/app-1-2-2-1 app-1-2-2-1=1pro/app-update
```

  - 기동되지 않는 App 업데이트 (RollingUpdate 테스트)
```bash
spec:
      containers:
        - name: app-1-2-2-1
          image: 1pro/app-error   # 수정
```
   - kubectl 명령으로 할 경우
```bash
[root@k8s-master ~]# kubectl set image -n default deployment/app-1-2-2-1 app-1-2-2-1=1pro/app-error
```
  - 업데이트 중지하고 롤백 할 경우
```bash
[root@k8s-master ~]# kubectl rollout undo -n default deployment/app-1-2-2-1
```

5. 강의에서 배포한 Object 삭제
```bash
[root@k8s-master ~]# kubectl delete -n default deploy app-1-2-2-1
[root@k8s-master ~]# kubectl delete -n default svc app-1-2-2-1
[root@k8s-master ~]# kubectl delete -n default hpa app-1-2-2-1
```
