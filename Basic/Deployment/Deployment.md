-----
### Deployment
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/9de6c144-c528-488f-821b-ad394c16a0f6">
</div>

1. 현재 한 서비스가 운영 중인데, 이 서비스를 업데이트 해야되서 재배포를 해야될 때 도움을 주는 Controller
2. 일반적으로 사용되는 업그레이드 방식
   - Recreate : Deployment를 만들면 V1의 Pod들이 생성
     + Pod 하나당 하나씩 자원이 사용된다고 가정
     + Recreate 방식으로 업그레이드를 하면, 먼저 기존 Pod들을 삭제 (이 때, 서비스에 대한 Down-Time이 발생되며, 자원 사용량도 없어지게 됨)
     + V2에 대한 Pod를 생성 (여기서는 2개)
     + 단점 : Down-Time이 발생하므로, 일시적 정지가 가능한 서비스일 경우에만 사용 가능한 방법

   - Rolling Update : 업그레이드를 실행하면 먼저 deployment는 V2 Pod를 하나 생성 (자원 사용량이 Pod 1개만큼 증가)
     + 이 상태부터는 V1, V2 모두 서비스되므로, V1, V2에 누군가가 접속
     + 그 다음 deployment는 V1의 Pod를 하나 삭제하고, 남은 V2 Pod를 하나 더 만들고, 남은 V1 Pod를 삭제하면서, 사용자들은 V2의 서비스에만 접속하게 됨
     + 배포 중간 추가적 자원을 요구하지만, 큰 장점은 Down-Time이 없다는 점

   - Blue-Greent : Deployment 자체로 제공되는 기능이 아닌, 이를 사용하는 것
     + Controller를 만들어서 Pod가 생성되면 Pod에는 Label이 있으므로, 서비스에 있는 selector와 연결
     + 운영되고 있는 상태에서 컨트롤러 하나를 더 생성하는데, V2 버전에 대한 Pod를 생성 (label 또한 V2)
     + 하지만, 이 때 자원 사용량은 2배가 됨
     + 이 때, Service에 있는 label만 변경하게 해주면, 기존 Pod와 관계는 끊어지고, V2 버전의 새 Pod들이 바로 연결되므로, Service에 대한 Down-Time이 없음
     + 그런데 만약 V2에 문제가 발생해 해당 Label만 V1로 바꿔주면, 기존 서비스로 전환되어 문제 시 RollBack이 쉬움
     + 상당히 많이 사용하고 안정적 배포 방식이지만, 단점은 자원이 2배가 필요

   - Canary : Canary와 같은 실험체를 통해 위험을 검증하고 위험이 없다는 게 확인되면 정식으로 배포하는 방식
     + V1에 해당하는 파드가 있고, Label이 붙어져 있는 상태에서 서비스를 만드는데, 이 V1이 아닌 type: web Label을 달아서 연결
     + 이렇게 운영 중인 상태에서 테스트용 컨트롤러를 만들 때, ReplicaSet을 1로 하여, V2에 대한 Pod를 하나 만들고, type: web Label을 달면 서비스에 연결
     + 이 서비스로 들어오는 트래픽 중 일부는 V2 버전으로 접근이 될 거고, 새 버전에 대한 테스트가 됨
     + 그러다가 문제가 발생하면 컨트롤러의 Replicas만 0으로 변경하면 됨
     + 즉, 불특정 다수에 대한 테스팅할 때 사용하는 방식
     + 다른 방식으로는 V1은 V1대로, V2는 V2대로 각 서비스를 만들고, Ingress Controller(유입되는 트래픽을 URL Path에 따라 서비스에 연결해주는 Controller)가 존재
       * Path 앞에 V2를 설정한 사람들은 새 버전에 대한 서비스를 사용
       * 즉, 특정 타겟을 정해놓고 테스팅 가능
       * 그렇게 해서 테스트 기간이 종료되고 문제가 없으면 V2에 대한 Pod 증가시킴
       * Ingress Controller의 설정을 변경한 후 기존 V1에 대한 내용을 삭제하면 됨
     + Down-Time이 존재하지 않고, 자원 사용량은 테스트 할 Pod 수나 V2 Pod를 얼마나 만들어 놓고 V1의 Pod를 다운시키느냐에 따라 필요한 자원량은 증가하게  
