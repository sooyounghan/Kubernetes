-----
### Argo Rollouts Deployment - Blue / Green, Canary
-----
1. Blue / Green 배포 : 운영 환경에서만 테스트가 가능한 상황에서 배포 시 롤백이 빠르며, V1과 V2 버전이 동시에 호출되는 구간이 없지만, 스크립트를 통해서 자동으로 진행되도록 할 수 있지만, V2에 과도한 트래픽이 유입될 수 있음
<div align="center">
<img src="https://github.com/user-attachments/assets/c04b63c0-e25b-4526-b7b5-2998a3bd9290" />
</div>

   - Kubernetes의 Resource들만 가지고 Blue / Green을 만들었을 때 동작을 보면 V1 Pod가 Service가 되고 있는 상태에서 V2 Deployment를 생성하고, Service를 붙여서 QA 담당자가 테스트를 한 후, 완료가 되면 기존 Service Selector를 V2로 바꿔서 트래픽 변경한 뒤, 문제가 없으면 V1 Deployment 삭제
   - Rollouts를 사용하면, Kubernetes에 Rollout Controller가 생성
     + Kubernetes 자체 리소스도 있지만, CRD라고 하여 누구나 Custom Resource를 정의해서 추가 가능
     + Kubernetes는 Go 언어로 작성되어서, Deployment를 생성하면 ReplicaSet이 생성되고, Replica 개수를 보고, Pod가 생성
     + 마찬가지로, 사용자가 CRD를 만든 뒤, Go 언어로 Resource가 어떻게 동작하라는 로직도 개발해야 함
     + 생성한 컨트롤러를 통해서 ReplicaSet이나 Pod 동작 변형이 가능한데, Rollouts가 이를 모두 구현

   - Service를 두 개 지정
     + 실제 서비스 사용자가 들어오는 Active 서비스
     + 업그레이드 중 V2 버전으로만 들어가 볼 수 있는 Preview 서비스
     + 이렇게 Resource를 생성 이후, Rollout은 ReplicaSet을 생성하며, 이후 Pod 생성
     + 두 Service Selector는 Pod에 매칭되게 설정해야하며, 연결이 됨 (현재는 굳이 Preview 서비스에 접속할 일이 없지만, 일단 두 Service 모두 V1 버전의 Pod로 트래픽 전달)
     + 따라서, 사용자들에게 서비스가 되고 있는 상태

   - Blue / Green 업그레이드 시작
     + Rollouts에서 태그 변경 : Argo Rollouts를 사용했다면, 싱크를 하거나 안쓰고 Kubernetes 자원을 직접 수정하면 업그레이드 동작이 시작
     + V2 버전의 ReplicaSet이 생성되고, Pod도 생성
     + Active Service는 그대로 V1 Pod를 보고 있지만, Preview Service는 V2 Pod로 연결이 변경 (여기서는 v2 버전의 Deployment 생성할 때, Label을 V2로 하고, QA용 서비스를 만들 때 Selector를 V2로 해서 수동으로 연결했지만, 여기서는 Rollout이 이 Pod들의 Label을 설정하고, 그 Label의 내용을 Service의 Selector에도 설정해 자동 연결)
     + 즉, ReplicaSet과 Pod가 매칭되는 원리를 Rollout에서 자동으로 동작
     + 따라서, QA 담당자는 Preview Service를 통해 테스트를 진행
     + 테스트가 완료되면 Promote라는 명령을 통해, 기존 Active Service는 순간적으로 V2 Pod로 트래픽이 변경되고, V1 Pod는 삭제

2. Canary : 특정 헤더 값에 따라 틀픽을 분기해줄 수 있는 구성으로, 배포를 만들기 전 Service를 별도로 만들지 않아도 특정 IP나 특정 사용자만 V2 호출 가능하며, 트래픽 배분을 조절하면서 V2 Pod에 트래픽을 넣을 수 있으므로, Cold Start를 방지하기 위해 쓰기도하며, 업그레이드 목적이 아닌 두 버전을 비교하기 위한 A-B 테스트 용도로 사용되기도 함
<div align="center">
<img src="https://github.com/user-attachments/assets/0e7a53ab-259e-4209-8e60-76ba36a2bc78" />
</div>

   - Deplomyent 동작을 보면, Ingress와 Ingress Controller인 Nginx가 필요
     + 서비스 사용자 트래픽은 Ingress Controller를 통해 들어오고 있으며, 업그레이드가 시작되면 V2 버전도 똑같은 리소스들이 생성
     + Ingress 내부 트래픽 가중치를 변경할 수 있는 옵션을 줄 수 있고, Nginx Controlle가 이를 보고 트래픽 분배
     + 따라서, 상황에 따라 가중치를 수정하면서 필요한 작업을 마치면 V1 버전에 관련된 리소스를 지우면 됨
     + Canary 배포는 Ingress는 Kubernetes 기본 리소스지만, Nginx는 별도 설치를 해야 하는 Add-On이므로, 이 솔루션이 없으면 기본 Kubernetes 리소스만으로는 구현 불가
    
   - Rollouts 사용
     + Nginx나 Istio 같은 Add-On이 있어야 섬세한 트래픽 조절이 가능
     + 동일한 Rollouts Controller를 생성
     + Deployment 배포 전략에 Canary 배포 전략으로 설정되었다면, Service를 하나 생성 후, 버전 변경 또는 저장이나 Argo CD에서 싱크를 하게 되면, V2 버전의 새 ReplicaSet이 생성
     + Rollouts 배포 전략으로 Canary를 선택 한 다음 상세 속성에 Steps라는 속성 존재
       * List 형식으로, setWeight와 pause를 번갈아가면서 설정
       * 현재 Pod가 2개 있고, setWeight를 33으로 줬으면, V2 Pod가 하나 생성 : Canary Pod
       * 따라서, setWeight는 전체 파트에서 Canary가 차지하는 Pod 비율이 33%가 되도록 하는 것
       * pause는 {}로 공백 : 이 상태에서 Promote 명령이 도착할 때까지 불안정 대기
       * 이 때, 필요한 테스트를 진행
       * 완료된 후, Promote 명령을 전송하면, 다음 단계로 setWeight: 66이므로 전체에서 V2 Pod의 비율이 66%가 되도록 Pod 수 조절
       * 따라서, V1 Pod 하나가 삭제되고, 새 V2 Pod가 하나 생성
       * Canary 테스트가 바로 완료되었다고 V2로 바로 전환하면 V2 Pod 부하가 심해질 수 있으므로 서서히 전환
       * 다음 스텝으로 넘어가서, pause로 { duration: 2m }이 있는데, 2분 동안 대기하므로, V1 Pod는 바로 삭제되지 않고, 2분이 지난 후 삭제

     + 즉, 이러한 단계에 따라 Canary 테스트를 하고, 배포 진행 과정을 Control할 수 있음 
