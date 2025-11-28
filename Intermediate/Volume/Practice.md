-----
### Volume - Dynamic Provisioning, StorageClass, Status, ReclaimPolicy
-----
: 해상 실습의 스토리지 솔루션은 Longhorn으로 대체
<div align="center">
<img src="https://github.com/user-attachments/assets/8f4f2d46-32e3-4807-9a72-f0cdfc22062a">
</div>

1. Longhorn 구축
   - 모든 노드(Master, Worker1, Worker2)에 iscsi 설치
```bash
yum --setopt=tsflags=noscripts install -y iscsi-initiator-utils
echo "InitiatorName=$(/sbin/iscsi-iname)" > /etc/iscsi/initiatorname.iscsi
systemctl enable iscsid
systemctl start iscsid
```

  - Longhorn 설치 및 설치 확인
```bash
// 설치
kubectl apply -f https://raw.githubusercontent.com/kubetm/kubetm.github.io/master/yamls/longhorn/longhorn-1.5.0.yaml

// Replica 수 줄이기
kubectl scale deploy -n longhorn-system csi-attacher --replicas=1
kubectl scale deploy -n longhorn-system csi-provisioner --replicas=1
kubectl scale deploy -n longhorn-system csi-resizer --replicas=1
kubectl scale deploy -n longhorn-system csi-snapshotter --replicas=1
kubectl scale deploy -n longhorn-system longhorn-ui --replicas=1

// 확인
kubectl get all -n longhorn-system
```

  - Dashboard 접속을 위한 Service 수정
```bash
// Service Type 변경 [ClusterIP -> NodePort]
kubectl edit svc -n longhorn-system longhorn-frontend
-----------
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: http
    nodePort: 30705  # port 번호 추가
  type: NodePort      # type 변경

// 확인 
kubectl get svc -n longhorn-system longhorn-frontend
-----------
NAME                TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
longhorn-frontend   NodePort   10.101.70.245   <none>        80:30705/TCP   8m18s
```
   - Dashboard 접속
```bash
http://192.168.56.30:30705
```
  
   - Fast StorageClass 추가 및 확인
```bash
// StorageClass 추가
kubectl apply -f - <<END
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: driver.longhorn.io
parameters:
  dataLocality: disabled
  fromBackup: ""
  fsType: ext4
  numberOfReplicas: "3"
  staleReplicaTimeout: "30"
END

// 확인
kubectl get sc
------------
NAME                 PROVISIONER          RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
fast                 driver.longhorn.io   Delete          Immediate           false                  34s
longhorn (default)   driver.longhorn.io   Delete          Immediate           true
```

2. Dynamic Provisioning
   - storageClassName="" : StorageClass는 사용 안됨 (이 때, PV의 실제 Volume은 바로 생성되지 않고, PVC에 Pod가 연결되어야 생성됨, Pod가 어느 Node에 할당될지를 확인 후 해당 Node에 hostPath 생성)
   - storageClassName=fast : 솔루션에서 제공하는 StorageClass 이름을 넣으면 자동으로 PV를 생성
   - PVC 생성시 storageClassName를 생략하면 Default로 설정된 StorageClass가 추가됨 (StorageClass로 PV생성시 Volume이 바로 생성)
   - PersistentVolume
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-hostpath1
spec:
  capacity:
    storage: 1G
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /mnt/hostpath
    type: DirectoryOrCreate
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-hostpath2
spec:
  capacity:
    storage: 2G
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /mnt/hostpath
    type: DirectoryOrCreate
```
   - StoragClass[""]를 사용하여 PersistentVolumeClaim 생성
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-hostpath1
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1G
  storageClassName: ""
```
   - StoragClass[fast]를 사용하여 PersistentVolumeClaim 생성
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-fast1
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1G
  storageClassName: "fast"
```
  - StoragClass[생략]를 사용하여 PersistentVolumeClaim 생성
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-default1
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2G
```

3. PV Status, ReclaimPolicy
   - PV Status
     + Available : PV 생성시 최초 상태
     + Bound : PVC와 연결 된 상태
     + Released : PVC가 삭제 된 상태
     + Failed : PV와 Volume의 연결이 끊긴 상태

   - ReclaimPolicy
     + Retain (default) : 데이터는 보존되나 재사용은 불가
     + Delete : StorageClass 사용시 Default로 PVC 삭제시 PV와 Volume 모두 삭제 

​   - Worker Node1에 Pod생성
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-hostpath1
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-worker1
  terminationGracePeriodSeconds: 0
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: hostpath
      mountPath: /mount1
  volumes:
  - name: hostpath
    persistentVolumeClaim:
      claimName: pvc-hostpath1
```
   - Pod(pod-hostpath1) 내부에 들어가서 파일 생성
```yaml
cd /mount1
touch file.txt
```
```bash
[root@pod-hostpath1 /]# cd /mount1
[root@pod-hostpath1 mount1]# touch file.txt
[root@pod-hostpath1 mount1]# ls
file.txt
```

   - Worker Node1의 동기화된 파일 확인
```yaml
cd /mnt/hostpath
ls
```
```bash
[root@k8s-worker1 mnt]# cd hostpath/
[root@k8s-worker1 hostpath]# ls
file.txt
```

   - 이후 Pod(pod-hostpath1) 및 PVC(pvc-hostpath1)를 삭제 후 PV(pv-hostpath1)의 ReclaimPolicy 상태 확인
   - PV(pv-hostpath1)의 ReclaimPolicy는 Released 상태로 변경됨
   - 이후 PV와 실제 Volume은 수동 삭제 필요
```bash
[root@k8s-worker1 mnt]# ls
hostpath
[root@k8s-worker1 mnt]# rm -rf hostpath/
[root@k8s-worker1 mnt]# ls
```

   - Released 된 상태의 PV는 재사용 하지 않음 = PVC 삭제는 곧 PV 삭제와 같음
   - 그리고 StorageClass로 만든 PVC(pvc-default1)를 삭제시 PV, Volume 확인
   - PV와 실제 Longhorn의 Volume은 자동 삭제됨
​
   - 실습 후 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 or Master Node에서 아래 명령 실행)
```bash
kubectl delete pv pv-hostpath1 pv-hostpath2
kubectl delete pvc pvc-fast1
```
  - Longhorn은 이후 계속 사용
  - kubectl (Force Deletion)
```bash
kubectl delete persistentvolumeclaims pvc-fast1 --namespace=default --grace-period 0 --force
kubectl delete persistentvolume pvc-b53fd802-3919-4fb0-8c1f-02221a3e4bc0 --grace-period 0 --force
```
