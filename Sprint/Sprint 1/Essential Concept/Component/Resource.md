-----
### 각 리소스별 동작 (Probe, Service, Secret, HPA)
-----
1. Pod 생성 및 Probe
<div align="center">
<img src="https://github.com/user-attachments/assets/f689d051-f290-4030-bd88-684277dde618" />
</div>

   - Deployment를 생성하면 kube-apiserver가 API를 받고, etcd를 통해 데이터를 저장시킴 : Deployment가 조회되거나 Dashboard에 나타나게 됨
     + kube-controller-manager는 계속 데이터베이스를 모니터링하며, Deployment가 조회되면 ReplicaSet 생성하라는 API를 전송
     + ReplicaSet이 만들어지면서 연결이 되고, 만들어진 ReplicaSet을 보고 Pod를 생성하라는 API를 전송
     + 따라서, Pod라는 데이터가 데이터베이스에 생성 (아직 데이터에만 존재한 것이며, 실제 컨테이너에 만들어진 것은 아님)

   - kube-scheduler는 보통 노드들에 대한 자원을 kube-apiserver를 통해 모니터링하고 있다가 데이터베이스에 Pod가 있는게 확인되면 Pod를 띄울 노드를 스케줄링
     + 이 때, Pod 안 사용자가 설정해 놓은 내용도 참고해서 마스터 노드를 스케줄링 하라는 내용이 업데이트 되면, kubelet 또한 kube-apiserver를 통해 자신의 노드 정보가 있는 Pod를 모니터링하고 있다가, 발견하고 Container Runtime에 Container 생성 요청
     + containerd가 컨테이너를 생성해줌
     + Probe 설정을 했다면, kubelet이 그 설정에 맞게 Container로 Health Check API를 주기적으로 전송

2. Service 동작
<div align="center">
<img src="https://github.com/user-attachments/assets/032fa4ae-f004-4c12-a99a-3d9a2dcd909c" />
</div>

   - nodePort 타입으로 Service를 만들어서 Pod를 연결 : kubelet이 kube-proxy한테 네트워크를 생성 요청
     + kube-proxy는 iptables에 내용을 추가 : 31231 Port가 들어오면, api-tester-1231 Service로 전송하라는 Mapping Rule 업데이트
     + iptables : Linux로 들어오는 모든 패킷을 관리하므로, 사용자가 API를 전송하면 컨테이너로 트래픽을 전달하주고, CNI로 설치했던 Calico가 처리

3. Secret 동작
<div align="center">
<img src="https://github.com/user-attachments/assets/bec00f8a-09eb-4522-b640-0c8aebda39eb" />
</div>

   - Container 내부의 파일들은 Node의 memory 영역으로 Mounting
     + 이 메모리는 전원이 꺼졌을 때, 지워지는 영역이므로 복구를 하더라도 일반 데이터처럼 살릴 수 없음
     + 또한, Kubernetes 입장에서 Memory에 Secret이 저장되면 Node의 메모리가 부족해짐
     + 추가적으로, 내용을 수정하게 되면, 바로 내용이 변경되지 않고, 주기적으로 Check를 하고 있다가, 변경사항이 생기면 업데이트를 해주므로 약간의 딜레이 발생 

4. HPA 동작
<div align="center">
<img src="https://github.com/user-attachments/assets/b16e372e-b433-46f8-a69b-ffe85b8b321a" />
</div>

   - 현재 컨테이너에 대한 자원 사용량은 Containerd가 알고 있음
   - kubelet은 CPU와 Memory를 10초에 한 번씩 조회
   - 이 상황에서 HPA에 metrics를 설정하고 deployment에 연결하면, Pod에 부하가 올라간다고 하더라도 아무런 일이 발생하지 않음
     + Metrics 서버를 별도로 설치를 해야 주기적으로 사용량을 60초 단위로 수집해서 저장 : 따라서, kube-controller-manager가 HPA 임계값과 Metrics을 15초 단위로 비교를 해보면서 스케일링 발생
     + 타이밍이 좋다면 바로 스케일링 동작, 최악의 경우 부하 발생 이후 스케일링이 되기까지 85초까지 소요
