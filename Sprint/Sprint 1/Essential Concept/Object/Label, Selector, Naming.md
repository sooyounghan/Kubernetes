-----
### Label / Selector / Naming
-----
1. Label / Selector / Naming (1) : Prometheus 예시
<div align="center">
<img src="https://github.com/user-attachments/assets/b51fc833-2d86-42fb-b072-d3170f190f0b" />
</div>

   - Prometheus를 설치하고 생성된 Pod의 Label 
     + Labeling : 대상의 간단한 정보를 붙여넣을 때 사용
     + Label : Object에 걸려있는 App 정보를 바로 파악하기 위해 사용하므로 Kubernetes에는 사용하는 것 권고

   - Prometheus 구성도
     + part-of : 애플리케이션 전체 이름 (전체 서비스를 대표하는 이름)
     + component : 서비스를 구성하고 있는 각각 분리된 기능
       * 예를 들어, prometheus는 Metric을 수집, 성능 정보에 대한 API 제공 / exporter는 Metric 제공 / Grafana는 Prometheus API를 통해 성능 정보를 시각화 해주는 기능
     + name : 애플리케이션의 실제 이름 (Metric을 제공하는 구성요소에는 여러 App이 존재할 수 있음)
       * exporter의 경우, kube-state-metrics는 Kubernetes Cluster에 Metric을 제공 / node-exporter : 노드, 즉 VM 정보를 제공하는 App
     + instance : 목적에 따라 여러 개 App을 설치 가능한 용도 (kube-prometheus 예하에 인스턴스를 식별하기 위해 k8s, k8s-1, k8s-2 등 생성 가능)
     + version : App 버전이 변경되면 수정 필요

   - Object Naming
     + 이름이 중복되면 안 되며, 특수문자는 - / .만 가능
     + Namespace : Namespace에 들어가는 App들의 범위를 나타내는 이름 사용
     + Object : Name과 Instance를 합쳐 생성 (즉, Object들의 이름은 Label의 Instance 이름)
     + ConfigMap : 뒤에 Rule이라는 단어 존재하는데, Pod에는 여러 ConfigMap 추가 가능 (환경 변수 및 외부에서 App에 전달하는 모든 데이터들을 ConfigMap에 저장 가능)
       * Prometheus에서는 ConfigMap에 성능 정보에 대한 계산 공식이 존재 : 기동할 때 초기 데이터로 사용하므로 이 목적에 맞게 rule이라는 단어를 붙임
       * 즉, 사용 목적에 따라 Naming 추가
     + Pod : Service를 여러 개 덧붙이기 가능
       * Cluster 내부에서 사용 : Instance 이름 뒤 internal 이름 부여하기도 함
       * 외부에서 접속 목적 : Instance 이름 뒤 external 이름 부여하기도 함
     + HPA도 임계치 설정에 따라 각 Object를 따로 만들면, 이름 부여 가능
     + Configmap의 경우도, App을 초기화할 때 별도로 사용하는 데이터가 있다면 이름을 부여하면 됨

   - Managed-by (Kubernetes 권고 Labeling)
     + 어떤 도구로 배포되었는지 Tool 이름을 작성
     + Object 생성 / 관리 주체를 알 수 있음

2. Deployment : Pod 업데이트를 관리
   - ReplicaSet : 복제본을 관리하는 역할 (Deployment Name - 임의 문자 추가)
   - Pod (ReplicaSet Name - 임의 문자 추가)

3. Label
   - App 정보 파악
   - Selector와 매칭되서 두 Object를 연결하는 데 사용 : Selector의 내용이 모두 Label에 포함되어야 함 (Label에 추가 정보가 더 있는 것은 괜찮지만, 역은 성립하지 않음)
     + Instance가 이름이 Unique 하므로, 이 자체만으로 Unique Pod Mapping이 가능
   - 즉, Deployment(Selector) - ReplicaSet(Label / Selctor) - Pod(Label) 연결 구성
     + Service(Selector) - Pod(Label) / PVC(Selector) - PV(Label)로 연결

   - 정보성 라벨링과 기능성 라벨을 하는 labels과 정보성 라벨링 역할을 하는 labels로 분류
     + 앞에 Prefix 부여 가능 : 도메인(Domain을 의미) [Optional]
     + 의미 있는 정보를 추가하여 생성 가능 

4. Kubernetes는 관계를 연결하는데 Object들 마다 다른 방법 제공
   - Selector - Label
   - Object 내에서 대상을 연결하는 속성 제공
      + 예) HPA에서 scaleTargetRef : Deployment 이름을 지정하는 속성
      + 예) Configmap과 Secret, PVC에서 Pod를 직접 연결하는 방법

5. 실제 생성되는 Object
<div align="center">
<img src="https://github.com/user-attachments/assets/fad6e1bd-7c02-4ddd-bd9d-ff158b51aa94" />
</div>

   - Deployment는 ReplicaSet과 연결
   - ReplicaSet, Service가 Pod와 연결
   - PVC는 PV와 연결
   - Node도 클러스터를 구성하면서 Kubernetes가 자동으로 새성한 Object가 존재하며, Label이 존재 : 이 Label은 Pod의 NodeSelecctor와 PV의 NodeAffinity와 연결

   - Namespace는 여러 App이 들어가므로, 두 개의 labels 속성만 부여
