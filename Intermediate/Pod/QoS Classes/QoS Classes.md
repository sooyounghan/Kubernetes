-----
### QoS Classes - Guaranteed, Burstable, BestEffort
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/4f22ba95-1ea8-48e1-b2e0-d6a84e004c35">
</div>

1. 사용 상황
   - 노드가 위 그림처럼 Resoucre가 있다고 가정 : 노드 위에 Pod가 3개 만들어져서 균등하게 자원을 모두 사용하고 있는 상황
   - 이 상태에서 Pod1이 추가적으로 자원을 더 사용해야 되는 상황이 발생
   - Kubnetes에서는 App의 중요도에 따라 관리할 수 있도록 3가지 단계로 Quality of Services (QoS))를 지원
     + 현재 이 상태에서는 BestEffort가 부여된 Pod가 가장 먼저 Down이 되어 자원이 반환
     + Pod 1은 필요한 양의 자원을 사용할 수 있게 됨
     + 노드에 어느 정도 자원이 남아잇지만, Pod2에서 그보다 많은 자원을 요구하는 상황이 됐다면, Burstable이 적용된 Pod가 다음으로 Down이 되면서 자원이 회수
     + Guaranteed가 마지막까지 Pod를 안정적으로 유지시켜주는 클래스

2. QoS는 Container에 Resource 설정이 있는데, requests와 limits에 따라 메모리와 CPU를 어떻게 설정할지 결정
   - Guaranteed : Pod에 여러 컨테이너가 있다면, 이 모든 컨테이너가 requests와 limits가 존재해야 하며, 그 안에 메모리와 CPU 설정이 둘 다 되어있어야 함
     + 그리고 각 컨테이너 안 설정된 메모리와 CPU 값은 requests, limits가 같아야 Kubernetes가 Pod를 Guaranteed 클래스로 판단
   - BestEffort : Pod의 어떤 컨테이너에도 requests와 limits가 설정되어 있지 않을 경우 클래스
   - Burestable : BestEffort와 Guaranteed의 중간 단계 클래스
     + requests와 limits는 설정되어 있지만, requests의 수치가 더 작다던가, 아니면 requests만 설정되어 있고 limits는 설정되어 있지 않은 경우
     + 또는 Pod에 컨테이너가 2개인데, 하나는 완벽하게 설정되었지만, 다른 하나는 설정되어 있지 않을 경우
   - 이 세가지 클래스는 결국 컨테이너의 requests와 limits을 어떻게 설정하느냐에 따라 달라짐

3. Burestable 등급에 여러 Pod가 있을 경우 삭제되는 순서
   - OOM(Out Of Memory) Score에 따라 결정
   - 두 Pod의 request memory가 5GB, 8GB로 설정, 안에 돌아가고 있는 App에 실제 메모리가 똑같이 4GB만큼 사용되었다고 가정한다면, Pod2의 메모리 사용량은 75%, Pod3는 50%
   - OOM Score는 Pod2가 75%로 더 크므로, Kubernetes는 OOM Score가 큰 Pod2를 먼저 제거
     
