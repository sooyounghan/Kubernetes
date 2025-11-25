-----
### Volume - emptyDir, hostPath, PV/PVC
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/418b0541-93bb-4421-956b-5568e78066f5">
</div>

1. emptyDir
   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-1
spec:
  containers:
  - name: container1
    image: kubetm/init
    volumeMounts:
    - name: empty-dir
      mountPath: /mount1
  - name: container2
    image: kubetm/init
    volumeMounts:
    - name: empty-dir
      mountPath: /mount2
  volumes:
  - name : empty-dir
    emptyDir: {}
```
<img width="833" height="148" alt="image" src="https://github.com/user-attachments/assets/df5b2d71-e6c8-4720-b532-dfa1f8b98e2b" />

   - Pod(container1) 내부로 들어가서 Mount 상태 확인 및 파일 생성
```bash
ls
mount | grep mount1
cd mount1
echo "file context" >> file.txt
```
```bash
[root@pod-volume-1 /]# ls
anaconda-post.log  bin  boot  dev  etc  home  lib  lib64  media  mnt  mount1  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@pod-volume-1 /]# mount | grep mount1
/dev/sda4 on /mount1 type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
[root@pod-volume-1 /]# cd mount1
[root@pod-volume-1 mount1]# ls
[root@pod-volume-1 mount1]# echo "file context" >> file.txt
```

   - Pod(container2) 내부로 들어가서 파일 동기화 확인
```bash
ls
cd mount2 
ls
cat file.txt
```
```bash
[root@pod-volume-1 /]# ls
anaconda-post.log  bin  boot  dev  etc  home  lib  lib64  media  mnt  mount1  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@pod-volume-1 /]# mount | grep mount1
/dev/sda4 on /mount1 type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
[root@pod-volume-1 /]# cd mount1
[root@pod-volume-1 mount1]# ls
[root@pod-volume-1 mount1]# echo "file context" >> file.txt
```

   - Pod 삭제 후 Pod 재생성 후 확인 : Pod 내에 존재하므로 Pod가 삭제되고, 다시 생성이 되면 소멸
```bash
[root@pod-volume-1 /]# ls
anaconda-post.log  bin  boot  dev  etc  home  lib  lib64  media  mnt  mount1  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@pod-volume-1 /]# cd mount1
[root@pod-volume-1 mount1]# ls
```

2. hostPath
   - 테스트 용도로 임시적으로 저장할 Volume이 필요 할 때 사용
   - HostPath 볼륨에는 보안 위험이 있기 때문에 운영환경에서는 가능하면 HostPath를 사용하지 않는 것을 권고
   - type 옵션 : DirectoryOrCreate (실제 경로가 없다면 생성), Directory (실제 경로가 있어야됨), FileOrCreate (실제 경로에 파일이 없다면 생성), File (실제 파일이 었어야함)
   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-2
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-worker1
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: host-path # Mount할 Volume의 이름
      mountPath: /mount1 # 컨테이너에서 접근할 Mount 경로
  volumes:
  - name : host-path
    hostPath:
      path: /node-v # HostPath의 경로
      type: DirectoryOrCreate # Path에 해당 노드가 없으면 Path를 직접 생성
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-3
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-worker1
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: host-path
      mountPath: /mount1
  volumes:
  - name : host-path
    hostPath:
      path: /node-v
      type: DirectoryOrCreate
```
   - Pod(pod-volume-2)에 들어가서 파일 생성
```bash
ls
cd mount1
echo "file context" >> file.txt
```
```bash
[root@pod-volume-2 /]# ls
anaconda-post.log  bin  boot  dev  etc  home  lib  lib64  media  mnt  mount1  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@pod-volume-2 /]# cd mount1
[root@pod-volume-2 mount1]# ls
[root@pod-volume-2 mount1]# echo "file context" >> file.txt
[root@pod-volume-2 mount1]# cat file.txt
file context
```

  - Pod(pod-volume-3)에 들어가서 동기화 확인
```bash
cd mount1
ls
```
```bash
[root@pod-volume-3 /]# ls
anaconda-post.log  bin  boot  dev  etc  home  lib  lib64  media  mnt  mount1  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@pod-volume-3 /]# cd mount1 
[root@pod-volume-3 mount1]# ls 
file.txt
```

  - Node(k8s-worker1)에 들어가서 동기화 확인
    + 접근 방법
<img width="623" height="775" alt="image" src="https://github.com/user-attachments/assets/b19b2abf-274d-4728-8bb6-21c52f28041f" />

```bash
cd /node-v
ls
```
```bash
[root@k8s-worker1 ~]# cd /
[root@k8s-worker1 /]# ls
bin   dev  home  lib64  mnt     opt   root  sbin  swapfile  tmp  var
boot  etc  lib   media  node-v  proc  run   srv   sys       usr
[root@k8s-worker1 /]# cd node-v
[root@k8s-worker1 node-v]# ls
file.txt
```
   - Pod 삭제 하고, 다시 만들더라도 Node 1에 HostPath는 그대로 있으므로 데이터는 유지
```bash
[root@pod-volume-3 /]# ls
anaconda-post.log  bin  boot  dev  etc  home  lib  lib64  media  mnt  mount1  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@pod-volume-3 /]# cd mount1
[root@pod-volume-3 mount1]# ls
file.txt
```

  - 노드를 지정하는 nodeSelector를 삭제하면, Schedular가 자원 상황을 파악하여 Node 2에 생성하지만, 다른 Node이므로 생성한 파일은 보이지 않음
```bash
[root@pod-volume-4 /]# ls
anaconda-post.log  bin  boot  dev  etc  home  lib  lib64  media  mnt  mount1  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@pod-volume-4 /]# cd mount1
```

3. PVC / PV
   - Pod가 삭제됐다 재생성되어도, PVC/PV를 통해 지정된 Volume에 연결됨
   - PVC와 PV는 capacity와 accessMode를 기준으로 연결됨 
      + 동일한 capacity와 accessMode가 많은 경우 Selector를 통해 PV와 PVC를 직접 연결 가능
   - PVC의 storageClassName: "" : PVC에 storageClassName를 사용하지 않고 PV에 연결하기 위한 설정 
   - PV의 local & nodeAffinity : 해당 PV에 연결된 Pod를 nodeAffinity에 지정한 Node에 스케줄링하며 해당 Node를 Volume으로 사용 (이 때 local-path에 지정된 경로가 해당 Node에 존재 해야함)
   - PersistentVolume
```yaml
apiVersion: v1
kind: PersistentVolume # 특정 물리적인 볼륨을 지정하는 PV 생성
metadata:
  name: pv-01
spec: 
  capacity:
    storage: 1G # 저장 용량 1GB
  accessModes:
  - ReadWriteOnce 
  local:
    path: /node-v 
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - {key: kubernetes.io/hostname, operator: In, values: [k8s-worker1]} # 해당 PV에 연결되는 Pod들은 Nod1에 생성
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-02
spec:
  capacity:
    storage: 1G
  accessModes:
  - ReadOnlyMany
  local:
    path: /node-v
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - {key: kubernetes.io/hostname, operator: In, values: [k8s-worker1]}
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-03
spec:
  capacity:
    storage: 2G
  accessModes:
  - ReadWriteOnce
  local:
    path: /node-v
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - {key: kubernetes.io/hostname, operator: In, values: [k8s-worker1]}
```
   - PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-01
spec: # 설정한 accessModes와 storage PV에 연
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1G
  storageClassName: ""
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-02
spec:
  accessModes:
  - ReadOnlyMany
  resources:
    requests:
      storage: 1G
  storageClassName: ""
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-03
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 3G
  storageClassName: ""
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-04
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1G
  storageClassName: ""
```
<img width="1230" height="246" alt="image" src="https://github.com/user-attachments/assets/7d57c89a-4e0e-491a-bf6a-fea448bb7778" />

   - 한 번 Binding이 적용되면, 해당 PV는 다른 PVC에서 사용 불가
   - pvc-3 PVC의 경우 요구된 용량과 적합한 Volume이 없으므로 Kubernetes가 연결을 못함
   - pv-03의 경우 요청은 1GB지만, 자신보다 높은 Storage가 있는 Volume에 할당

   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-5
spec:
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: pvc-pv
      mountPath: /mount3 # 해당 Volume이 Container에 접근할 때 mount3 Path에 연
  volumes:
  - name : pvc-pv
    persistentVolumeClaim: # 볼륨 생성 시 PVC 사
      claimName: pvc-01 # pvc-01 사용
```

   - Pod(pod-volume-5)에 들어가서 파일 생성
```bash
ls
cd mount3
ls
```
```bash
[root@pod-volume-5 /]# ls
anaconda-post.log  bin  boot  dev  etc  home  lib  lib64  media  mnt  mount3  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@pod-volume-5 /]# cd /mount3
[root@pod-volume-5 mount3]# ls
file.txt
```

   - 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 또는 Master Node에서 아래 명령 실행)
```bash
kubectl delete pod pod-volume-1 pod-volume-2 pod-volume-3 pod-volume-4 pod-volume-5
kubectl delete pvc pv-01 pvc-02 pvc-03
kubectl delete pv pv-01 pv-02 pv-03 pv-04
```
   - 생성시 [PV -> PVC -> Pod] 순으로, 삭제시 [Pod -> PVC -> PV] 순으로 삭제해야 함

4. PV/PVC를 label과 selector를 이용해 연결하는 방법
   - PersistentVolume
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-05
  labels:
    pv: pv-05
spec:
  capacity:
    storage: 1G
  accessModes:
  - ReadWriteOnce
  local:
    path: /node-v
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - {key: kubernetes.io/hostname, operator: In, values: [k8s-worker1]}
```
   - PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-05
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1G
  storageClassName: ""
  selector:
    matchLabels:
      pv: pv-05
```
