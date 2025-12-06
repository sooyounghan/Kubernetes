-----
### PV, PVC (local, hostPath)
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/83845cf1-6eef-44c5-9570-5d05f10bd0bd" />
</div>

1. Pod의 volumes에 PVC를 연결
   - 필수값 : resoucre, accessModes
   - Pod의 volumes과 PVC과 연결
   - PVC(selector)와 PV(labels) 연결 : 두 속성의 내용이 같지 않으면 연결되지 않음

2. local
   - 사용하면 필수로 nodeAffinity라는 속성을 꼭 사용해야 함 : 어느 노드에 Pod를 생성할 것인지 정하는 속성
   - Master Node 생성 시에 기본적으로 Node라는 Object가 생성되고, labels(kubernetes.io/hostname: k8s-master)가 부여
     + 여러 Wokrer Node를 만들면 동일하게 규칙에 따라 생성
     + 💡 nodeAffinity라는 속성이 있는 PV에 연결된 Pod는 설정한 Node 위에 생성
     + mountPath(/usr/src/mypp/files/dev)를 설정하면, Kubernetes가 이를 Node와 연결 (/root/k8s-local-volume/1231)
     + 💡 즉, local의 값은 Master Node에 연결할 Path
   - 근본적으로 PVC와 PV를 사용하여 별도 스토리지 공간을 만들어놔야하는 이유는 App이 컨테이너 안에 있다가 파일을 저장하면, Pod가 죽을 때 파일도 같이 삭제되기 때문에, 구성해 놓은 Path에 파일을 만들면, 실제 파일은 해당 Path에 저장되므로 Pod가 죽더라도 데이터가 유지될 수 있음
     + 따라서, Kubernetes가 기존 설정한 Path를 연결하고 새로 만들어진 App일지라도, 기존 파일들을 그대로 볼 수 있음

3. 테스트 환경에서 많이 사용하는 방식 : PV의 volumes의 hostPath 사용
   - 똑같이 Path를 지정할 수 있음
   - Pod의 nodeSelector를 설정하면 Node 지정 가능
   - 주의사항 : 가능하면 사용하지 않는 것이 좋음 (테스트용으로 사용하기 편하지만, 원래는 노드에 있는 정보를 App에 조회하는 용도로 사용하는 것이 좋음)
     + Grafana의 Loki를 통해 모든 App의 Log를 봤는데, 실제 Pod들의 Log는 /var/log/pods로 저장되어 있고, Loki의 Promtail라는 Pod가 hostPath로 /var/log/pod를 조회하고 있으므로, 모든 Pod들의 Log들을 Loki를 통해 확인 가능
     + 따라서, 필요한 파일 또는 디렉토리 범위만 지정하고 ReadOnly로 Mounting해서 사용해야 함

   - 따라서, 운영환경에서는 저장 용도로 사용하면 안 되며, 노드의 저장 공간은 한정되어 있으므로, 특정 Pod에서 저장 공간을 사용하다가 Node의 디스크 저장 공간이 부족해서 노드가 죽어버리면, 다른 Pod들도 같이 죽게 됨
   - 하지만, 모든 Node에 같은 경로로 Path를 만들고 NAS로 NFS 연결
     + 노드에 디스크를 쓰는 것이 아닌 NAS 서버만 관리할 수 있지만, 수동으로 연결해야 되는 문제가 생기므로, 자동화 운영 관점에서 좋지 않음

4. PVC와 PV 사용 이유
   - Pod를 만드는 주체는 개발자
   - Volume Solution에는 여러 가지고 있고, 각 기능에 따라 써야 되는 속성도 달라지므로, 개발자가 Pod를 만들면서 신경 쓸 수 없음
   - 따라서, 인프라 담당자가 이걸 관리하는 컨셉으로 PV라는 Object 생성
   - PVC는 Pod에서 필요한 자원을 요청하는 용도로 개발자가 만드는데, 인터페이스 역할을 해주는 역할로, PV의 Solution이 변경되더라도 Pod를 수정하지 않도록 개발

5. 동작 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/9eace3fc-5256-47d6-a5d7-6e1b97302d15" />
</div>

   - local 동작 확인
```bash
﻿// 1번 API - 파일 생성
http://192.168.56.30:31231/create-file-pod
http://192.168.56.30:31231/create-file-pv
```

```bash
// 2번 - Container 임시 폴더 확인
[root@k8s-master ~]# kubectl exec -n anotherclass-123 -it <pod-name> -- ls /usr/src/myapp/tmp
// 2번 - Container 영구저장 폴더 확인
[root@k8s-master ~]# kubectl exec -n anotherclass-123 -it <pod-name> -- ls /usr/src/myapp/files/dev
// 2번 - master node 폴더 확인
[root@k8s-master ~]# ls /root/k8s-local-volume/1231
```

```bash
// 3번 - Pod 삭제
[root@k8s-master ~]# kubectl delete -n anotherclass-123 pod <pod-name>
```

```bash
// 4번 API - 파일 조회
http://192.168.56.30:31231/list-file-pod
http://192.168.56.30:31231/list-file-pv
```

2. hostPath 동작 확인 - Deployment 수정 후 [1~4] 실행
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: anotherclass-123
  name: api-tester-1231
spec:
  template:
    spec:
      nodeSelector:
        kubernetes.io/hostname: k8s-master
      containers:
        - name: api-tester-1231
          volumeMounts:
            - name: files
              mountPath: /usr/src/myapp/files/dev
            - name: secret-datasource
              mountPath: /usr/src/myapp/datasource
      volumes:
        - name: files
          persistentVolumeClaim:  // 삭제
            claimName: api-tester-1231-files  // 삭제
          // 아래 hostPath 추가
          hostPath:
            path: /root/k8s-local-volume/1231
        - name: secret-datasource
          secret:
            secretName: api-tester-1231-postgresql
```
