-----
### Volume - emptyDir, hostPath, PV & PVC
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/dded4259-5e50-420f-bae0-eb986e4db2f8">
</div>

1. emptyDir
   - 컨테이너들이 서로 데이터를 공유하기 위해 볼륨을 사용하는 것
   - 최초 이 Volume이 생성될 때는 항상 이 Volume 안에 비어있으므로 emptyDir라는 명칭이 됨
   - Pod안에 생성 : Pod에 문제가 발생하여 재생성되면 데이터 전부 소실
      + 따라서, emptyDir에 쓰이는 데이터는 꼭 일시적인 사용 목적에 의한 데이터만 넣는 것이 좋음
   - yaml 파일 내용
     + 두 개의 Container가 존재하는데, 둘 다 Volume을 Mount (Container가 Path로 Volume을 연결하는 의미)
     + Container1 : mountPath가 /mount1 Path
     + Container2 : mountPath가 /mount2 Path
     + Volume Mount Path가 다르더라도, name: empty-dir로 똑같은 Volume을 지정하고 있으므로 컨테이너마다 자신이 원하는 경로를 사용하고 있지만, 결국 한 Volume을 Mount
     + volumes: (Volume의 속성) - emptyDir: [] 명시 

2. hostPath
   - Pod들이 적재된 노드에 Path를 Volume으로 사용
   - emptyDir와 다른 점은 이 Path를 각각의 Pod들이 마운트되서 공유하므로, Pod가 소멸되도 노드에 있는 데이터는 사라지지 않음
   - 문제점 : 만약 위 그림에서 Pod2가 죽어서 재생성이 될 때, 꼭 해당 노드에 재생성이 될 보장이 없음
     + 만약 재생성되는 순간에 Schedular가 자원 상황을 보고 Node2에 Pod를 만들 수 있음
     + 또한 Node 1에 장애가 생겨서 다른 Node에 Pod이 옮겨질 수 있는데, 이렇게 다른 노드로 Pod가 옮겨진 경우 원래 있던 볼륨을 Mount할 수 없게 됨 (HostPath이므로, 자신의 Poath가 올라가져 있는 해당 노드만 Volume 사용 가능)
     + 따라서, Node가 추가 될 때마다 똑같은 이름의 경로를 만들어서 직접 노드에 있는 Path끼리 Mount를 시켜줘야 함 (운영자가 직접 Linux 시스템 별도의 Mount 기술을 사용해서 연결해야 함)
   - yaml 파일 내용
     + Pod 생성 시, Container에 Volume을 Mount 할 건데, Path는 /mount1이며, 이 Path에 대한 HostPath라는 이름은 host-path
     + Volume에 대한 hostPath의 Path는 /node-v, type은 Directory
     + 주의사항 : Host에 있는 경로는 Pod가 만들어지기 전 사전에 만들어져 있어야 함

3. PV (Persistent Volume) & PVC (Persistent Volume Claim)
   - 영속성 있는 볼륨을 제공하기 위한 개념
   - 실제 볼륨들의 형태는 다양함 : Local Volume, 외부에 원격으로 사용되는 형태의 Volume
   - Amazon이나 Git에 연결 가능하며, NFS를 사용해 다른 서버와 연결 가능
   - Storage OS 같이 볼륨을 직접 만들고 관리할 수 있는 솔루션들을 PV와 정의하고 연결
   - Pod는 PV에 바로 연결하지 않고, Persisten Volume Claim을 통해서 PV와 연결이 됨
     + Kubernetes는 Volume 사용에 있어서 User 영역(Pod의 서비스를 만들고 배포를 관리하는 서비스 담당자)과 Admin 영역(Kubernetes를 담당하는 운영자)을 구분
   - yaml 파일 내용
     + 각 볼륨에 따라 연결하는 속성이 다름
     + Pod를 생성할 때는 PVC 이름을 연결하여, Volume을 만들면 이 Volume을 컨테이너에서 사용
     + PV의 경우, matchExpression에 지정한 Node 위에만 Pod가 생성 : Pod가 재생성될 때마다 해당 지정 노드 위에 만들어지므로 데이터 영속적인 입장에서 문제는 없음
     + capacity와 accessModes를 지정해놓으면, 이를 통해 PVC가 PV를 연결할 때 Kubernetes가 자동으로 연결
