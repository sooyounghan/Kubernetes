-----
### Ingress
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/055236a1-6239-4c72-938e-13a53f656ac6">
</div>

1. 사용 목적 : Service Load Balancing
   - 만약 쇼핑몰을 운영 중이라고 가정했을 때, 쇼핑 페이지와 고객센터 그리고 주문 서비스를 Pod 별로 생성
     + 이렇게 애플리케이션을 나누면 쇼핑 페이지가 문제가 생겨도 고객 응대나 주문 관리를 하는 데는 영향이 없다는 장점이 존재
     + 외부에서 연결할 수 있도록 각각 서비스를 달아주고, 사용자들로 하여금 쇼핑 페이지에 접근하기 위해 ```www.mall.com```이라는 도메인 이름으로, 고객 센터에 접근하려면 ```www.mall.com/customer```, 주문 서비스에 접근하려고 한다면 ```www.mall.com/order```로 설정
       * 이렇게 하려고 한다면, 각 Path에 따라 서비스 IP를 이어줄 수 있는 L4 이나 L7 스위치라는 장비가 있어야 함
   - Kubernetes에는 Ingress라는 Object가 해당 역할을 대신해줌
     + 이 도메인에 Root Path가 들어오면, Shopping Pod로 연결
     + /customer가 있다면, Customer Pod가 있는 서비스로 연결
     + /order가 있다면, Order Pod로 연결
     + 따라서, 별도의 IP 로드 밸런싱을 해주는 장비가 필요 없음

2. 사용 예 : Canary Upgrade
   - V1의 App들이 구동되서 서비스가 되고 있는 상태에서 테스트 할 버전 V2의 App을 구도시키고 Ingress를 만들어서 두 버전의 서비스를 연결하면, Ingress를 통해 접근을 했을 때, 10%의 일부 트래픽만 V2로 가도록 설정 가능
   - 해당 % 수치를 변경하거나 연결이 될 때, 들어있는 헤더 값 별로 트래픽 조정도 가능

<div align="center">
<img src="https://github.com/user-attachments/assets/fe793e79-e8df-4384-9069-032baa20fb77">
</div>

3. Kubernetes에서 Ingress 동작 과정
   - Ingress라는 Object는 Kubernetes가 설치가 되었으면 바로 생성 가능
   - Ingress에는 Host로 Domain Name을 넣을 수 있고, 이 도메인으로 들어오는 트래픽은 Path에 따라 원하는 서비스로 연결을 하라는 내용이 주 설정
   - 해당 Rule을 실행할 구현체를 만들어야 함 : Kubernetes에서 이 구현체를 만들기 위해 별도 플러그인을 설치해야 함
     + Ingress Controller 라고도 하며, 대표적으로 Nginx이나 Kong이 있으며, 그 외에 많은 컨트롤러도 존재
     + 만약, Nginx를 설치하게 되면, Nginx에 대한 Namespace가 생기고, 이 위에 Deployment와 Replicaset이 만들어지며, 실제 Ingress의 구현체인 Nginx Pod가 생성
     + 그럼 Pod가 Ingress Rule이 있는지 보고 있다면, Role대로 서비스를 연결해주는 역할을 함
     + 따라서, 이 Rule에 따라 서비스를 연결해주는 역할을 함

   - 트래픽은 Nginx Pod를 거쳐야 하므로 외부에서 접근할 수 있는 서비스 하나를 만들어서 해당 Pod에 연결을 해줘야 함
   - 직접 Kubernetes를 설치했다면, NodePort를 만들어서 외부에 연결할 수 있을 것이고, 클라우드 서비스를 이용하고 있다면 Load Banalcer를 만들어 외부에 연결 가능
   - 따라서, Ingress Rule에서 지정한 도메인으로 접근이 오면, 이 서비스를 통해 Nginx Pod로 트래픽이 들어와서 Ingress Rule에 따라 지정된 서비스와 Pod에 접근 가능
   - 또한, Ingress를 추가할 수 있는데, 다른 도메인을 주고 Path없이 바로 Service로 연결할 수 있음
     + 현재는 Nginx를 사용하므로, 바로 Ingress가 인식이 되어 지정된 서비스에 연결
     + 사용자가 Ingress에 지정된 도메인 이름으로 들어오면, 해당 서비스에 연결이 되고 여러 개를 만들 수 있음

<div align="center">
<img src="https://github.com/user-attachments/assets/ab66bf9e-6aae-4ea3-a02e-9a91a9cef5f4">
</div>

4. Ingress의 기능
   - Service LoadBalancing
     + 각 업무 별로 Pod와 Service를 생성하며, 사전에 Ngnix가 설치되어있고, 이 Pod의 외부에서 연결이 되도록 Node Port 서비스도 연결되어 있으므로 마스터의 호스트 IP인 ```192.168.0.30:30431```로 접근하면 Pod의 80번 Port로 트래픽이 전송
     + 환경이 구성된 상태에서 Ingress를 생성하고, 규칙을 각 Path에 따라 의도하는 서비스로 매칭을 시켜주면 됨
     + 사용자가 ```192.168.0.30:30431```로 들어오면 서비스 페이지로 접근이 되고 svc-order Path로 접근하면 주문 관리 페이지로 접근

   - Canary Upgrade
     + ```www.app.com```이라는 도메인 이름으로 사용자가 접근하면 svc-v1이라는 이름의 서비스로 연결이 되는 구성
     + 현재 사용자들에게 App이 운영되고 있는 상황으로, Canary Upgrade를 테스트할 Pod와 Service를 띄움
     + Ingress를 하나 더 만드는데 호스트 이름은 ```www.app.com```로 동일하고, Service Name을 svc-v2로 주면 연결고리가 만들어짐
     + 새로 만든 Ingress에 @weight: 10%라는 Annotation을 설정 : 해당 도메인으로 접근하는 트래픽의 10%은 svc-v2 Pod로 진입해 테스팅
     + 그리고, 특정 나라별로 테스트를 하고 싶을 때는 헤더의 옵션을 이용하면 사이트에 접근하는 언어가 KR일 때, 100% 트래픽으로 v2 Pod로 접근
     + 이 밖에도 Ingress에 Annotation을 적용하면, 다양한 기능들을 사용할 수 있음

   - Https
     + Ingress를 통해서 HTTPS로 연결할 수 있도록 인증서 관리도 할 수 있음 : Pod 자체에서 인증서 기능을 제공하기 힘들 때 이용하면 좋음
     + 기본적으로 구성된 상태에서 Nginx Pod로 HTTPS를 사용하려면 443 Port를 연결해야 함
     + Ingress를 만들 때, 호스트 도메인 이름과 서비스를 연결을 하고, TLS라는 옵션이 존재하는데, 여기에 secretName으로 실제 Secret Object를 연결하고, Secret 안에는 데이터 값을 인증서로 저장하고 있는데, 이렇게 구성하면 사용자가 도메인 이름 앞에 https를 붙여야만 접근 가능
  
