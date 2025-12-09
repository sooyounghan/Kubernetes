-----
### 배포 전략을 세울 때 고려해야 하는 요소
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/7d2680c3-2173-41fc-85fe-64c231530f52" />
</div>

1. ReCreate 
   - 버전 1이 작동하는 상태에서 버전 2로 배포를 하려면, 기존 Pod를 먼저 삭제시키기 떄문에 Downtime 발생
   - V2 버전의 새로운 Pod가 생성되면서 서비스와 연결이 되면, 다시 서비스가 활성화
   - 배포 툴 : kubectl, Helm, Kustomize

2. RollingUpdate
   - V1가 아직 서비스가 되고 있는 상태에서 V2를 생성하므로 버전 1과 버전 2 Pod가 동시에 호출되는 구간이 있다가 V2 Pod로 모두 전환되는 배포 방식
   - 배포 툴 : kubectl, Helm, Kustomize

3. 두 전략 모두 Kubernetes에서 Deployment 기능을 제공하므로 Depolyment만 업데이트 시키면 자동 배포
   - 배포 중간에 잠깐 정지 후 에러가 나면 Rollback 가능
   - 하지만, 트래픽에 대한 제어는 불가능 : 서비스에 연결된 Pod들은 1 / N을 통해 트래픽을 골고루 받게 됨
   - 데이터베이스 스키마 변경 시 새 배포가 되어야 하므로, DB 작업이 완료가 된 뒤 V2을 올려야 하므로, 결국 수동으로 작업을 할 수 밖에 없음
   - + 작업 순서는 서비스 중단 공지를 한 후 Deployment의 Replicas를 0으로 만들어서 서비스를 내림
     + DB 작업이 끝나면 deployment의 image 태그를 v2로 변경하고 Replicas를 2로 수정
     + 따라서, App에 어떤 배포가 적합다고 결정되더라도 다양한 상황에 따라 맞는 배포 전략 고려 필요
       
4. Blue / Green : 운영 DB에서만 테스트가 가능한 경우 사용하는 것이 좋음
   - Deployment에서 제공해주는 기능이 아님
   - 현재 Service와 Pod는 selector와 labels에 의해서 연결이 되어 있으며, V2 버전의 Deployment를 하나 더 생성 (label은 V2)
     + Service의 Selector를 V1에서 V2로 수정하면 바로 트래픽이 V2로만 전송되게 됨
     + 즉, V2 Depolyment를 생성하고 Service의 Selector를 변경한 다음, V1 Deployment 삭제
   - Kubenetes가 나오기 전 부터 오랫동안 기존 운영 환경에서 배포를 해왔던 방식 : 평소 L4라는 네트워크 장비에 VM 2대를 연결해놓고 이중화시켜서 사용
     + 배포를 위해서는 네트워크 담당자에게 L4의 VM2의 연결 해제 요청
     + V2에다가 소스 패치 후 가동 시킨 뒤, L4 담당자에게 트래픽 전환
     + L4 담당자는 V1의 연결을 먼저 끊고, V2를 연결하면 ReCreate / V2를 연결하고 VM을 끊으면 RollingUpdate
     + V1을 끊는 것과 동시에 VM2를 연결시켜놓는 방식으로 트래픽 전환 : 이후, VM1도 패치를 한 뒤 이중화
  - 수동 배포 시에 Roll-Back이 빠름 : V2로 서비스를 전환한 다음에, Log를 보다가 문제를 발견하면 바로 V1으로 Roll-Back 가능
  - 스크립트를 통해 자동 배포 가능 : 즉, ReCreate와 Rolling Update의 단점들을 모두 보완한 자동 배포
  - 주의사항 : 트래픽 전환 시에 V2에 과도한 트래픽이 갑자기 유입되면 문제 발생 가능
    + 처음 트래픽이 들어오면 App 내부적으로 메모리나 DB Connection들을 최적화시키느라 CPU를 많이 사용하는 경우 발생
    + 그러다 어느 정도 시간이 지나면, CPU가 안정화되는 데 트래픽이 많은 시점에 Blue / Green 배포를 하게 되면 Service가 바로 뻗어버리는 현상을 볼 수 있음
    + 따라서, 사전에 부하 테스트를 해봐서 App이 어느 정도 트래픽 상에서 배포를 해도 문제가 없는지 확인하는 것이 좋음
  - 배포 툴 : Jenkins, kubectl, ArgoCD

5. Canary : 특정 헤더 값에 한해서만 (특정 트래픽만) V2로 트래픽 유입 (SourceIP, User, Language)
   - Blue / Greent과 배포랑 구성은 동일
   - V2 Pod에도 새 서비스를 만드는데, QA용이 아닌 배포용으로 사용
   - Ingress Controller인 Ngnix와 각 Service에 Ingress라는 리소스르 만들어줌 : 트래픽량 조절 가능
     + Nginx는 사전 설치되었다고 가정하면, V2의 Deployment와 Ingress, Service를 한 번에 생성
     + Ingress 안에 가중치를 변경하는 부분이 존재하는데, 이 수치를 변경해나가면서 Canary 배포 시작
     + V2로 트래픽 전환이 모두 종료되면 V1에 관련된 리소스 모두 삭제
   - 해당 배포 전략을 사용하면 10%의 트래픽만 V2로 유입시켜 보면서 새 버전에 문제가 없는지 모니터링 가능
   - 콜드 스타드 방지 : 10%의 트래픽만 보내므로 V2에 문제가 생겨도 과도한 트래픽이 한 번에 들어가는 것이 아니므로 App이 충분한 Warm-Up을 할 수 있는 시간을 가질 수 있음
     + 또한 메모리와 DB 커넥션 상태가 최적화가 된 다음 나머지 트래픽이 전송되므로, 가동하자마자 다량의 트래픽이 주입되는 콜드 스타드 방지 가능
   - 두 버전을 비교 가능
     + 그 목적이 V1에서 V2로 전환을 하려는 것만이 아닌, 두 버전을 열어 놓고 사용자들이 어떤 버전의 반응이 더 좋은지를 비교를 하다가 A보다 B가 더 좋으면 B로 전환을 하는 것이고, 다시 A를 쓰는 것
     + A / B 테스트 : 배포 전략이 아닌, Canary 포 상황에서 V1과 V2를 비교하기 위한 테스트 방법론 중 하나
   - 배포 툴 : ArgoCD, Nginx, Istio
