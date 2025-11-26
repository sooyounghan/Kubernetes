-----
### Life Cycle
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/b5dc0f83-fad6-4252-8cf1-e862639c5071">
</div>

1. Pod의 Life Cycle
   - 각 단계에 따라 행해지는 행동이 다르며, Pod 역시 해당 단계에 따라 주요 행동이 있음
<div align="center">
<img src="https://github.com/user-attachments/assets/4d0b1f7d-8e58-48a4-91af-5b04df423d88">
</div>

   - Pod를 생성한 후 내용 하단을 보면, Status라는 부분이 존재
   - Pod 내에 status 안에 phase가 존재 : Pod의 전체 상태를 대표하는 속성
   - Pod가 생성되면서 실행되는 단계들이 있는데, 그 단계와 상태를 알려주는 것이 conditions
   - Pod 안에는 Container가 존재하므로, 컨테이너마다도 Status가 있으므로 각 컨테이너를 대표하는 상태인 containerStatuses가 존재
   - Pod 상태 : Pending, Running, Succeed, Failed, Unknown 5개의 상태 존재
   - Conditions : Initialized, PodScheduled, ContainerReady, Ready가 존재
     + Reason : Conditions의 세부 내용을 알려주는 항목 (ContainersNotReady, PodCompleted)
     + status에 true, false가 존재하며 false의 경우 그 원인이 무엇인지 알아야하므로 reason이 존재
   - containerStatues를 보면 state에 Waiting, Running, Terminated 세 가지 상태 존재하며, 세부 내용을 알기 위한 reason 존재
     + 현재 컨테이너는 Waiting 중이며, 이유는 Container Creating
   - Pod의 imageID가 없으므로 아직 이미지가 다운로드 되지 않을 것을 알 수 있음

<div align="center">
<img src="https://github.com/user-attachments/assets/2a54a11b-50b9-470f-bd18-ffd5c59b9e2e">
</div>

2. Pod의 상태
   - 최초 Pod 상태는 Pending
     + Initalized
       * initContainer : 본 컨테이너가 기동되기 전 초기화시켜야 할 내용들이 있을 경우 그 내용을 담는 컨테이너
       * 만약 Volume이나 보안 세팅을 위해 사전 설정을 해야 되는 일이 있다면, Pod 생성 내용 안에 initContainer 항목으로 초기화 스크립트를 넣을 수 있음
       * 이 스크립트가 본 컨테이너보다 먼저 실행이 되어 그 작업이 성공적으로 끝났거나 아예 설정을 하지 않았을 경우 true 또는 false로 반환
     + PodScheduled (Node Scheduling)
       * Pod가 어느 노드 위에 올라갈지 직접 노드를 지정했을 때 지정한 노드 또는 Kubernetes가 자원의 상황을 판단해 노드를 결정하기도 하는데, 완료가 되면 true
     + 동작 순서 : PodScheduled → Initialized
     + 다음으로 컨테이너 이미지를 다운로드 하는 동작 존재 : 두 과정 동안에 컨테이너 상태는 Waiting, Reason은 Container Creating

   - 본격적으로 컨테이너가 기동되면 Pod와 Container 상태는 Running
     + 일반적으로 정상적 기동이 될 수 있지만, 하나 또는 모든 컨테이너가 기동 중 문제가 발생해 재시작될 수 있음 : 이 때 컨테이너의 상태는 Waiting이 되고, CrashLoopBackOff라는 reason이 나오게 됨
     + Kubernetes는 이러한 상태들에 대해 Running으로 간주하되, 내부 컨디션의 ContainerReady와 Ready는 False
     + 결국 모든 컨테이너들이 정상적으로 기동이 되어서 원활하게 돌아간다면 컨티션들은 모두 true로 변경
     + 따라서, 일반적으로 계속 서비스가 운영이 되어야 하는 경우 이 status를 계속 유지해야 함
     + 💡 중요 : Pod의 상태가 Running이더라도, 내부 컨테이너 상태가 Running이 아닐 수 있으므로, Pod뿐만 아니라 컨테이너 상태도 모니터링해야 함
     + Job, CronJob으로 생성된 Pod의 경우, 자신의 일을 수행했을 때는 Running 중이지만, 일을 마치게 되면 Pod는 더 이상 일을 하지 않는 상태가 되어 Pod의 상태는 Failed가 되며, 컨테이너들이 모두 Completed로 해당 일을 잘 마치면 Succeed
       * 이 때, 또 Pod의 Condition 값이 변하게 되는데, 성공이건 실패건 간에 ContainerReady와 Ready의 값이 False로 변경

   - Pending 중에 바로 Failed로 변경되는 경우도 있고, Pending이나 Running 중 통신 장애가 발생하면 Pod가 Unknown 상태로 바뀜 : 통신 장애가 빨리 해결 되면, 다시 기준 상태로 변경되지만, 계속 지속이 되면 Failed로 변경
     
