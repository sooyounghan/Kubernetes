-----
### Service - Headless, Endpoint, ExternalName
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/04cff872-3541-4b5c-940f-2cbb2ebf614c">
</div>

1. 공유기를 통해 192.168번대로 시작하는 IP로 내부망이 형성되어있다고 가정
   - 3대의 서버를 이용해 Master와 Node로 Kubernetes Cluster가 형성
     + 클러스터 안에는 Pod를 위한 IP 대역과 Service를 위한 IP 대역이 존재
     + 10번과 20번으로 시작하는 IP 대역은 Kubernetes를 구성할 때 설정할 수 있는 부분으로, 그림과 같이 설정했다면 Pod가 생성이 되면 20번대의 IP 할당, Service가 만들어지면 10번으로 시작하는 IP가 자동 할당
     + 또한, 이 둘은 연결되어 있는 상태이며, 이 상태에서 Service가 클러스터 IP로 만든 서비스라면, 서비스에 접근 Kubernetes를 구성하고 있는 서버에서만 호출이 가능 (내부망에 IP를 할당받은 여러 기기들이 있을 수 있지만, 직접 Service의 IP 호출 불가)
     + 내부망에 있는 다른 서버들이 접근하기 위해서는 NodePort라는 Service를 만들면, Kubernetes를 구성하는 서버들에게 30000번 대 포트가 생성이 되고, 해당 Port가 Service와 연결
     + 따라서, Kubernetes 관리자는 내부망에 있는 사람들에게 이 서버들 중 하나의 IP와 Port를 알려주면, 이를 통해 내부망에 있는 사람들은 해당 서비스에 연결될 수 있음
   - Google Cloud, AWS, Azure와 같이 Cloud Provider를 사용해 Kubernetes를 구축했을 경우
     + 이러한 경우 Kubernetes에서 Load Balancer 타입의 서버를 만들을 때, NodePort가 생성되면서 이 Port에 Node Balancer가 연결이 되고, 외부망에 있는 이용자들은 해당 IP를 통해 서비스에 접근 가능

<div align="center">
<img src="https://github.com/user-attachments/assets/5cb00be4-1214-4e94-be8b-33613d4248ae">
</div>

2. 사용자 관점
   - Kubernetes 망에 있는 Service에 궁극적으로 연결되어 있는 Pod에 접근하기 위해 Cluster IP, Node Port, Load Balancer 타입 Service를 생성
   - 사용자의 접근의 경우 Service가 만들어진 후 IP를 확인하고, 해당 IP로 접근하면 되는데, Pod의 경우 이 자원들이 동시에 배포될 수 있음
     + 따라서, Pod A가 Pod B에 접근해야 되는데, Pod A IP는 Pod나 서비스가 생성 시 동적 할당 되므로, 미리 IP를 알 수 없음
     + 또한, Pod B가 문제가 생겨서 죽게 되면 자동으로 재생성되므로 IP가 변경될 수 있으므로, Pod A가 IP를 알더라도 계속 쓸 수 없음
     + 이런 문제 해결을 위해 DNS와 Headless Service가 필요
     + ExternalName을 통해 외부 연결을 Pod 수정 없이 (Pod가 외부 특정 사이트에 접근 해서 데이터를 가져오는 상황에서 접근 주소를 변경해야 될 경우, 경로를 변경하기 위해 Pod를 수정하고 재배포 하지 않아도 됨) 변경할 수 있음

<div align="center">
<img src="https://github.com/user-attachments/assets/92689467-c610-409b-af01-31e2a752435f">
</div>

3. Kubernetes Cluster 안에는 DNS 서버가 별도로 존재
   - DNS 서버에는 서비스의 도메인 이름과 IP가 저장되어 있어 Pod가 Server 1에 대한 도메인 이름을 질의하면 해당 IP를 알려줌
   - 그리고 내부망에서도 DNS 서버가 구축되어 있다면, 내부 서버들이 생겼을 때, 해당 이름들이 DNS로 등록되어 있을 것이며, Pod가 User 1을 찾을 때, Kubernetes DNS에 없다면 DNS 메커니즘 상 상위 DNS를 찾게 되고 해당 이름의 IP를 알려줌
   - 마찬가지로 외부에 있는 사이트도 도메인 이름이 외부 DNS로 등록되어 있으므로, Google을 찾으면, 부모의 부모를 찾아 결국 Google IP 주소를 획득
   - 따라서, Pod의 IP 주소를 몰라도 DNS 서비스 이름으로 IP를 질의하여 연결 가능

<div align="center">
<img src="https://github.com/user-attachments/assets/a2b7e5d7-7ef5-4982-9fde-bd524cc646d9">
</div>

4. Pod 1이나 Pod 2를 선택해서 연결을 하고 싶을 때는, Pod의 Headless 서비스를 연결하게 되면, DNS 서버에 Pod 이름과 서비스 이름이 붙여겨서 도메인 이름으로 등록
   - 따라서, Pod 입장에서는 Pod 1에 접근하기 위해 IP 주소를 알 필요가 없고, 도메인 이름으로 접근 가능
  
<div align="center">
<img src="https://github.com/user-attachments/assets/2313de69-db52-497f-89c3-97644005762f">
</div>
5. ExternalName 서비스는 특정 외부 도메인 주소를 넣을 수 있음
   - Google 도메인 이름을 넣으면 DNS를 거쳐서 Google IP를 가져올 수 있고, Pod는 이 서비스를 통해 데이터를 가져오도록 한다면, 추후 데이터를 GitHub에 가져오도록 변경하면, Pod 수정 없이 서비스의 ExternalName만 변경해주면 됨

<div align="center">
<img src="https://github.com/user-attachments/assets/f8e064fc-c775-49ba-93cc-655fdba2ca30">
</div>

6. Headless Service
   - Default라는 네임스페이스에 Pod 2개와 Service가 연결
     + 이 서비스는 클러스터 IP로 만든 서비스로, IP도 각각 동적으로 할당
   - Kubernetes DNS : kubernetes.default.svc.cluster.local로, 이 DNS는 Pod 또는 Service를 생성하면 긴 도메인 이름과 IP가 저장
     + 이름의 구조
       * 서비스의 경우 서비스 이름.네임스페이스.svc.DNS 이름이 생성
       * Pod의 경우 해당 IP.네임스페이스.pod.DNS 이름 생성
       * 이렇게 규칙을 만들어서 생성되는 것을 Fully Qualified Domain Name이라해서 FQDN이라고 부름
       * 같은 네임 스페이스 안에서는 서비스는 앞자리만 짧게 써도 되지만, Pod는 다 입력해야 함 (앞부분이 IP이므로 쓸 수 가 없음)
       * Pod의 입장에서 서비스 도메인 이름을 DNS 질의를 통해 IP를 가져오기 때문에, 서비스 이름만 알아도 해당 서비스에 접근할 수 있고, 모든 이름들은 사용자가 직접 만드는 것이므로 미리 서비스의 이름을 예상해 Pod에 저장 가능

   - 따라서, 단순히 서비스를 연결하는 데 클러스터 IP로 서비스를 만들어도 문제가 없음
   - 하지만, Pod가 Pod 4에 직접 연결을 하고 싶다면, Service를 Headless 서비스로 만들어야 함
   - + 만드는 방법은 clusterIP: None으로 설정 (서비스의 IP를 만들지 않겠다는 내용으로, 실제로 생성되지 않음)
     + 💡 Pod를 생성할 때는 hostname이라는 속성에 도메인 이름을 넣어야 하고, subdomain에 해당 서비스의 이름을 넣어줘야 함
     + DNS에 서비스는 클러스터 IP 생성과 동일하지만, 다른 점은 서비스의 IP가 없으므로, 서비스 이름을 호출하게 되면 연결되어 있는 모든 Pod들의 IP를 전달
     + Pod들을 보면 Pod의 hostName.serviceName.네임스페이스.svc로 생성
     + 따라서, Pod는 이름들을 미리 정해놓고 DNS를 통해서 원하는 Pod로 직접 통신할 수 있음

<div align="center">
<img src="https://github.com/user-attachments/assets/fb20a1ee-246c-462a-a11d-42ade53f112d">
</div>

7. EndPoint
   - 서비스와 Pod을 연결할 때 Label을 통해 연결 : 이는 사용자 측면에서 둘의 연결을 위한 도구일 뿐, Kubernetes는 매칭이 됐을 때, EndPoint를 생성해 실제 연결
   - 엔드포인트는 서비스의 이름과 동일한 이름으로 EndPoint 이름을 설정하고, EndPoint 안에는 Pod의 접속 정보를 넣어줌
   - Label과 Selector 없이도 EndPoint를 직접 만들어, 서비스의 이름과 Pod의 IP 정보를 넣게 되면 똑같이 연결 (즉, 서비스 연결 대상을 사용자가 직접 지정)
   - IP와 Port가 내부 Port를 가리킬 수 있고, 외부 IP 주소를 안다면 외부를 가키리 수 있음
     + 하지만, IP는 변경 가능성이 존재하므로 Domain 이름을 사용하는 것이므로, 이러한 Domain 이름을 지정할 때 필요한 것이 ExternalName

8. ExternalName
   - 해당 속성을 통해 도메인 이름을 넣을 수 있음
   - DNS 캐시가 내부와 외부 DNS를 찾아 IP를 알아내며, Pod는 서비스를 가리키고만 있으면, 서비스에서 필요시마다 해당 도메인 주소를 변경할 수 있어 접속할 곳이 변경되더라도 Pod를 수정하고 재배포하는 일은 없어지게 됨
