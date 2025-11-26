-----
### Controller
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/c07cd365-d7ad-4535-8f16-c34672e4f1b3">
</div>

1. Kubernetes에는 여러 컨트롤러가 존재
2. Kubernetes의 Controller는 서비스를 관리하고 운영하는데 큰 도움을 줌
   - Auto-Healing : 노드 위 Pod가 갑자기 다운 또는 Pod가 스케줄링 되어 있는 Node가 다운되면 Pod에서 진행되던 서비스에 장애가 발생하는데 이를 즉각적으로 인지하고 Pod를 다른 Node에 만들어주는 기능
   - Auto-Scaling : Pod의 ResourceLimit 상태가 되었을 때, 컨트롤러가 이 상태를 파악하고 Pod를 하나 더 만들어줌으로써, 부하를 분산시키고 Pod가 다운되지 않도록 해줌으로써, 서비스가 성능에 대한 장애 없이 안정적 서비스를 운영하게 해주는 기능
   - Software Upgrate : 여러 Pod에 대한 버전을 업그레이드 해야 될 경우 컨트롤러를 통해 한 번에 쉽게 할 수 있으며, 업그레이드 도중 문제가 생기면 Rollback 할 수 있는 기능도 제공
   - Job : 일시적인 작업을 해야될 경우 컨트롤러가 필요한 순간에만 Pod를 만들어서 해당 작업을 이행하고 삭제함으로, 그 순간에만 자원이 사용되고 작업 후 다시 반환되어 효율적 자원 활용이 가능해짐
