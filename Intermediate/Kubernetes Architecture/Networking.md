-----
### Network - Pod / Service Network (Calico), Pause Container
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/7195a29a-3540-41a0-bdd8-e8cb12b3867c">
</div>

1. Networking Architecture
   - Kubernetes를 이루는 Master와 Worker Node들이 있고, Pod 네트워킹에 대한 영역 존재
     + pod-network-cidr : 네트워크 대역에 대해 영역을 설정한 부분
   - 다른 Pod가 생성됐을 때, 이 두 Pod 간의 통신은 Kubernetes에서 노드마다 설치가 되는 네트워크 플러그인에 의해 통
   - Kubernetes에서 기본적으로 제공하는 kube-proxy라는 네트워크 플러그인이 있는데, 네트워크 기능이 많이 제한적이므로 잘 사용하지 않고, CNI라는 네트워크 인터페이스가 있는데, 이를 통해 다양한 오픈소스 네트워크 플러그인들을 설치 가능
     + Calico : 네트워크 플러그인으로, 네트워크 플러그인이 하는 역할은 노드 위에 Pod들 간 통신과 외부 네트워크를 통한 타 Node 위에 Pod들 간의 통신을 담당

2. Service Networking
   - Kubernetes를 설치할 때 별도 옵션을 주지 않아 Default 값으로 IP 대역이 설정
   - Pod에 서비스를 붙이게 되면, 서비스도 고유 IP가 생성이 되고, 이 서비스가 생성된 동시에 kube-dns에 서비스 이름과 IP에 대한 도메인이 등록
     + api-server가 Wokrer Node들마다 Pod의 형태로 띄워져 있는 kube-proxy에 이 서비스의 IP가 어느 IP와 연결되어있는지 정보를 보내줌
     + 여기서 서비스 Service IP를 Pod IP로 바꾸는 NAT 기능이 필요한데, kube-proxy가 ipyables이나 IPVS를 어떻게 이용하느냐에 따라 Proxy Mode라고 하는 작동 방식이 세 가지 존재
     + 이 상태에서 Pod가 Service 이름을 호출하게 되면 kube-dns를 통해 IP를 알게 되고, 얻은 Service IP를 NAT 영역으로 추가하게 되는데, 여기에 해당 Service에 대한 Pod 맵핑 정보가 있으므로 트래픽은 네트워크 플러그인을 통해 해당 Pod로 가게 됨
   - 결국 Service Object의 실체는 이 NAT 영역 내 설정이고, Service를 삭제하게 되면 다시 API 서버가 이걸 감지해서 kube-proxy한테 이 설정을 지우라고 요청

<div align="center">
<img src="https://github.com/user-attachments/assets/dcf52d91-1867-49f8-b43a-d1fb9f63caa0">
</div>

2. Pod Network (with Calico)
   - Pod 네트워크를 담당하는 Pause Container와 Cluster Network를 담당하는 네트워크 플러그인으로 구성
   - Pause Container
     + Pod를 생성하게 되면, 생성한 컨테이너 외에도 Pause Container는 네트워킹을 담당하는 컨테이너가 자동으로 생성
     + 이 컨테이너에 인터페이스가 있으며, IP도 생기는데, Kubernetes가 이 Pause Contaniner의 네트워크 Namespace를 Pod 내 모든 컨테이너들한테 같이 쓰도록 구성
     + 따라서, IP에 대한 네트워크를 같이 공유하게 되고, 컨테이너 간 구분은 Port를 통해 할 수 있음
     + 한편, 이 Worker Node에 Host Name Space가 있고, 여기에 이 호스트 IP 인터페이스가 있는데, Pause 컨테이너가 생기면, 이 Host Name Namespace에 가상 인터페이스가 하나 생기면서 Pause Container의 Interface와 연결
     + Pod를 만들 떄마다 가상 인터페이스가 생겨서 1:1로 매칭되며, Worker Node에서 IP와 Port를 전송하면 트래픽이 흐르게 됨

   - Network Plugin
     + CNI를 설치하지 않고, Kubernetes의 기본 네트워크인 Calico을 사용했을 때의 구성
     + 기본적인 네트워크 구성에 대해서는 Pod를 생성하게 되면 새로운 네트워크 인터페이스를 만드는데, Pod 네트워크에 대한 부분과 Host 자체 네트워크로 구성
     + 따라서, Pod 인터페이스와 Host Netowrk의 가상 인터페이스가 연결이 되며, 네트워크 플러그인인 kubenet이 하는 역할은 가상 네트워크를 cbr0라는 컨테이너 브릿지에 포함을 시켜줘야, 이 Bridge Network 대역은 Pod 네트워크 대역을 참고해서 이보다 낮은 단계 대역으로 설정이 되며, Bridge 내 생성되는 Pod IP는 Bridge CIDR 범위 내에서 생성
     + Bridge에 세팅된 CIDR를 보면, 총 255개의 IP를 할당할 수 있으며, 한 Node 위에 그 이상은 생성 불가 : kubenet을 썼을 때, 단점 중 하나이며, 이렇게 Bridge를 통해 Pod 간의 통신이 가능해지는 것
     + Router는 NAT 기능을 통해 기능이 제공이 되고, IP가 Pod IP 대역이면 해당 트래픽은 Bridge 쪽으로 내려주고, 그 외의 IP는 위로 올려줌

<div align="center">
<img src="https://github.com/user-attachments/assets/8bacee57-0b97-4921-8f12-d0b2deb2c1a7">
</div>

   - Calico CNI를 사용하면, Host Network에 생성된 가상 인터페이스가 Router에 바로 연결이 되는 구조이므로, 두 Pod 통신은 라우터가 담당해주므로 CIDR은 kubenet보다 더 큰 범위를 가지고 있어, 한 Node에서 Pod한테 더 많은 IP 부여 가능
     + 라우터 윗단에는 Overlay 네트워크를 제공해주는 층이 존재하며, 이 종류에는 IPIP 방식과 VXLAN이라는 방식이 존재
     + Overlay Network는 이 노드 위 있는 Pod가 타 노드에 있는 Pod와 통신할 수 있도록 해주는 것
     + 만약, Pod D에서 Pod B로 트래픽을 전송한다고 했을 때, Pod D에서 Pod B의 IP인 ```20.111.156.7```로 호출을 하면, 그대로 라우터에 있는 가상 인터페이스를 지나 해당 IP는 이 라우터 내에 없으므로 Overlay 네트워크 층까지 올라감
     + 그리고 Calicao는 Pod의 IP 대역이 어느 노드에 있는지 알고 있으므로, Overlay 네트워크 층에서 패킷을 해당 노드의 IP로 변경해주고, 실제 Pod의 IP 안에 숨겨져 있음 (Pod의 IP가 Encapsulation되었다고 표현)
     + 이 트래픽은 22번 Host 인터페이스로 가게 되고, Node 1번으로 들어온 트래픽이 이 노드의 Overlay 네트워크 층에서 Decapsulation 되면서 원래의 Pod IP로 변환
     + 이 IP 대역에 있는 라우터로 트래픽이 전송되고, 라우터 안에서 해당 IP에 맞는 가상 인터페이스를 지나 최종적으로 요청한 Pod까지 도착하게 되는 것

3. Service Network
   - Proxy Mode : Kubernetes가 제공하는 Service Network의 Proxy Mode에는 세 종류가 있음
     + Pod를 만들고, 서비스를 설정하는데 Pod가 정상적으로 기동된 상태라면 중간에 Endpoint라는 Object가 생성이 되서 실제 연결 상태를 담당
     + 서비스의 IP는 서비스 네트워크 CIDR 범위 내에서 생성이 되며, api-server는 Endpoint를 감시하고 있다가, 모든 노드 위에 DaemonSet으로 설치되어 있는 kube-proxy한테 이 서비스의 IP는 Pod의 IP로 Forwarding 된다는 정보 제공
   - userspace Mode : Linux Wokrer 노드에 기본적으로 설치되어 있는 IP 테이블에 서비스 CIDR로 들어오는 트래픽은 모두 kube-proxy한테 주도록 설정이 되어 있는 상태로, Pod에서 서비스 IP를 호출할 경우 트래픽은 IP 테이블을 거쳐 kube-proxy에게 가게 됨
     + kube-proxy가 자신이 가지고 있는 맵핑 정보를 보고 트래픽을 Pod IP로 변경해주며, Pod IP로 변경된 트래픽은 다음부터 Pod 네트워크 영역으로 통신되서 트래픽이 전달
     + 단점 : 구조를 보면 모든 트래픽이 kube-proxy를 반드시 자나야 하는 구조로, 모든 트래픽을 감당하기엔 kube-proxy 자체에 대한 성능이나 안정성이 좋지 안항 쓰이지 않는 방식
   - iptables Mode : kube-proxy가 IP 맵핑 정보를 IP Table에 직접 등록하는 방식이 아닌 Pod가 보내는 서비스 IP는 IP Tables에서 직접 Pod IP로 변환
     + Kubernetes를 설치했을 때 기본 모드이기도 하고, 성능이나 안정성이 훨씬 좋음
   - ipvs Mode : Linux에서는 IPVS라고 하며 L4 Load-Balancer라는 것을 제공하는데, 서비스 모드를 IPVS 모드로 설정하면 IP 테이블과 같은 역할을 함
     + 낮은 부하 상태에서는 네트워킹 성능은 비슷하지만, 부하가 커질수록 IPVS 성능이 더 좋음

<div align="center">
<img src="https://github.com/user-attachments/assets/49794b3a-f4f7-4f04-9068-0cb4b5d89def">
</div>

4. Calico 플러그인 사용 시 서비스 네트워크 부분
   - 서비스를 만들 떄 클러스터 IP나 Node IP 타입에 따라 트래픽 흐름이 다른데, 먼저 클러스터 IP 타입의 서비스를 만들었을 때, Calico Pod 네트워크 구조
   - Route 부분에 서비스 IP를 Pod IP로 변환해주는 NAT 역할을 하는 기능이 존재
   - 그리고 Pod A에서 Pod B로 트래픽을 날린다고 했을 때, Pod B에 클러스터 IP 타입 서비스를 연결하며, Pod 뒤에서 서비스 IP로 트래픽이 전송되면, NAT에서 해당 서비스 IP와 매칭되는 Pod IP로 변환
   - 그 다음부터는 Overlay 네트워크에서 IP 캡슐화되고, Pod 네트워크 단과 통신하게 됨
     
<div align="center">
<img src="https://github.com/user-attachments/assets/cb69ff89-14c2-4012-9bcd-ece26b41c8ff">
</div>

5. NodePort 타입의 서비스를 연결하면 모든 Node에 있는 kube-proxy가 자신의 노드의 30000번 대의 NodePort를 열어주며, 외부에서 이 Host IP의 Port로 트래픽이 들어오게 되면, iptables에서 이 트래픽을 Calico 네트워크 플러그인으로 전송
   - 여기서부터는 클러스터 IP와 같이 NAT 기능을 통해 해당 IP로 변환이 되면서 Pod 네트워크 영역으로 넘어감
