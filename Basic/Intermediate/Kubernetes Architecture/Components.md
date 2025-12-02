-----
### Pod Creation
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/c43f1aed-4367-445c-9cc0-f6a166bad1cc">
</div>

1. Pod가 생성되는 과정
   - 먼저 마스터 노드에는 etcd와 kube-scheduler, kube-apiserver가 존재
     + 일반적인 설치를 했을 때, 이 컴포넌트들은 Pod 형태로 띄워져서 구동 중인 상태
     + 마스터 노드에 /etc/kubernetes/manifest라는 폴더에 컴포넌트 생성에 대한 yaml 파일들이 존재
     + Kubernetes가 기동 시에, 이 파일들을 읽어서 Pod들을 Static으로 띄움

   - Wokrer Node들은 Kubernetes가 설치할 때 같이 설치한 kubelet과 Container Runtime 도구인 Dokcer가 구동 중인 상태
   - kubectl 명령으로 Pod 생성 요청을 날렸다고 가정
     + Pod 생성 명령은 kube-apiserver로 전달이 되고, 이 서버는 이 Pod에 대한 입력 정보를 저장해둠
     + etcd는 Kuberentes에서 여러 데이터들을 저장하는 DB 역할을 함
     + kube-scheduler는 수시로 각각의 노드의 자원들을 확인
     + 따라서, 만약 현재 상태가 Node 2에 컨테이너가 들어가고 있어서 자원이 사용되고 있다는 것을 알 수 있으며, 하는 일이 watch라는 기능으로 kube-apiserver를 통해 etcd에 Pod 생성 요청이 들어온 게 있는지 감시
     + 이렇게 Pod 생성 요청 하나 있는 것을 발견하면, 현재 노드 자원 상태를 확인하고 이 Pod가 어느 노드로 가는 것이 좋을 지 판단해 Pod에 노드 정보를 붙여줌
     + 각각의 노드에 kubelet을 보면 kube-apiserver에 watch라는 게 설정되어있고, Pod에 자신이 노드 정보가 붙어있는지 확인을 하고 자신의 노드가 붙어있는 Pod임을 발견하면, 이 정보를 가져와서 Pod를 생성하기 시작
   - kubelet이 Pod를 생성하는데, 크게 두 가지 일을 함
     + 먼저 Docker에게 Container를 만들라고 요청하면, Docker는 바로 컨테이너를 생성
     + /etc/kubernetes/manifest에 kube-proxy.yaml이라는 Controller를 생성하는 yaml 파일이 존재하고, 이 파일은 DaemonSet이라 모든 노드에 kube-proxy Pod가 생성되어 있음 : 이 상태에서 kubelet은 kube-proxy에게 네트워크 생성 요청을 하고, kube-proxy가 새로 생성된 컨테이너에 통신이 되도록 도와줌
     
-----
### Deployment Creation
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/d8fd2e23-061b-4df0-a149-592b1f4f50d1">
</div>

1. Deployment를 생성하는 예제
2. 다른 컴포넌트들과 마찬가지로 /etc/kubernetes/manifest라는 폴더에 kube-controller-manager.yam 파일이 있어서 Controller Manger Pod가 띄워져 있고, 이 안에서 컨트롤러들에 대한 기능이 각 쓰레드 형태로 작동
3. 이 상태에서 사용자가 kubectl create 명령으로 deployment replicas라는 명령을 줘서 생성
   + 명령어가 kube-apiserver로 전달이 되고, etcd DB에 저장
   + 한편 컨트롤러 매니저의 Deployment 스레드는 kube-apiserver한테 deployment 관련 정보가 들어오면 알려달라고 watch를 걸어둔 상태인데, deployment가 진이했으므로, deployment 스레드가 이 내용을 읽고 ReplicaSet을 만들라고 요청
   + ReplicaSet이 ReplicaSet Object가 있는지 watch를 설정한 상태라는 걸 감지하고 ReplicaSet 안에 ReplicaSet이 몇 개 있는지 확인한 다음 그만큼 Pod를 만들라고 요청
   + 이 상황에서 kube-scheduler에는 노드가 할당되지 않는 Pod를 가지하고 현재 노드들의 자원을 고려해 Pod들힌테 스케줄링 될 노드를 할당해주고, 각 노드에 있는 kubelet은 자신에게 할당된 Pod들을 감지하고 Pod 안에 컨테이너 내용을 도커에게 생성 요청
   + 도커가 컨테이너를 만들어주고, kubelet이 kube-proxy를 통해 네트워크를 연결
