-----
### Logging - PLG Stack
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/c6b52240-6c71-4af3-9703-ffab52df6110">
</div>

1. Kubernetes Architecture Logging : App의 Log 데이터를 의미하며, 모니터링은 CPU나 메모리와 같은 자원을 이야기하는 것
   - Kubernetes가 자체적으로 어느 정도 제공되는 Core Pipeline과 추가적으로 플러그인을 설치해서 기능을 더 강화할 수 있는 Service Pipleline이 존재
   - Core Pipleline
     + Worker Node에는 kubelet과 Container Runtime 영역이 존재
     + 노드의 자원인 CPU, 메모리와 데이터를 저장하는 디스크가 존재
     + Kuberenetes로 Pod 생성 요청을 받으면, 해당 노드 위에 존재하는 kubelet은 Pod를 생성하고 그 안에 있는 컨테이너 생성은 컨테이너 런타임에게 위임 (여러 종류 컨테이너 런타임을 만들어주는 솔루션이 존재하지만, 현재 Docker로 진행)
     + Worker Node의 디스크를 사용하면서 데이터를 쓰고 로그를 생성하며, 서비스가 기동되면서 CPU와 메모리 자원도 사용
     + 각 노드에 있는 kubelent은 별도로 설치되어 있는 Resource 예측기인 cAdvisor를 통해 CPU 메모리 정보를 가져올 수 있으며, 각 노드의 모니터링 정보는 metrics-server로 전송되고, 사용자는 kubectl top 명령으로 kube-apiserver를 통해 정보를 조회하여 kubelet을 통해 해당 컨테이너의 kublet log 파일을 볼 수 있음

   - Service Pipleline
     + kubelet처럼 모든 노드에 설치가 되어서 데이터를 가져오는 역할을 위해 DaemonSet을 이용한 Agent 영역이 존재하는데, Container Runtime 혹은 Wokrer Node를 통해 Log와 Resource 자원을 수집하는 역할을 함
     + 그리고 별도의 Worker Node에 Metric 서버와 같이 각 노드에 Metric을 수집하고 분석하는 서버 영역(Metric Aggregator / Analytics)이 존재 : 많은 데이터가 저장되므로 별도의 저장소(Metrics Store)를 구성하는 것이 좋음
     + Web UI가 수집 서버로 쿼를 하면서 필요한 정보를 사용자에게 보여줌
     + 대표적인 오픈소스 : Elastic Search, Loki, Prometheus가 존재

<div align="center">
<img src="https://github.com/user-attachments/assets/9e5e5c50-0b90-4e31-91bc-45328483a8c2">
</div>

2. Logging Archtitecture
   - Node-Level Logging : 단일 노드 안에서 일어나는 로깅 구조
     + 단일 노드에서 로그 파일이 생기는 구조를 보면, kubelet과 Docker, Worker Node 영역이 존재
     + Pod 내 컨테이너에서 App 자체 로그를 /log/app/app.log 파일 형태로 생성한다고 가정 : 컨테이너는 도커가 만드므로, 도커의 Log Driver를 통해 로그가 생성되는데, /etc/docker의 daemon.json 파일에 존재
       * 컨테이너 당 최대 3개의 로그 파일이 생기고, 각 파일은 100MB 씩 저장되도록 하는 설정이며, 그 이상 만들어지면 이전 파일들이 지워짐
       * Docker 설정 상태에서 단순히 App에 로그 파일을 쌓는다고, 도커에 로깅 파일로 생기는 것이 아닌, 로그를 stdout, stderr로 출력해야 하며, 출력된 로그는 /var/lib/docker/containers라는 폴더에 컨테이너 별 ID가 생성되어 ```<container-id>.json```이라는 로그 파일 생성 (즉, Docker 자체적인 로깅 드라이버를 통해 컨테이너 안의 로그가 워커 노드에 만들어지는 과정)
     + Kubernetes는 /var/log/pod라는 폴더를 생성하고, 네이밍 규칙(```<Name-Space>_<Pod_Name>_<Pod-ID>/<Container-Name>/0.log```)을 가지고 파일을 Link하며, 이 상태에서 Pod가 삭제되면 컨테이너도 삭제되어 더 이상 로그를 볼 수 없음
     + 즉, 노드 레벨 로깅은 Pod가 살아있는 동안 유지가 되는 거고, 이 때 kubectl log 명령으로 Pod의 현재 실시간 로그를 조회해볼 수 있는데, Pod가 죽으면 더 이상 볼 수 없으므로, 클러스터 레벨 로깅이 필요
     + /dev/termination-log : App에 오류가 생기거나 LivenessProbe 문제 시 Pod가 Restart 되는데, 원인이 되는 로그를 보기 위해 로그 파일을 확인하는데, 그럴 필요 없이 죽기 전에 Error Log를 해당 파일에 쓰게 되면, Kuberenetes가 이 파일을 읽어 Pod의 상세 Status 부분에 표시를 해줌으로, Pod 디테일 조회 명령으로 Restart의 원인이 되는 로그를 볼 수 있게 해줌
     + 또한, App의 로그 뿐만 아니라 Pod가 Restart 또는 Scale-Out 등의 Pod의 상태를 볼 수 있음
   - Cluster-Level Logging : Kubernetes 클러스터 내 모든 노드들에 있는 로그들을 포괄하는 로깅 (Kubernetes에서 기능 제공이 아닌 아키텍쳐만 제시하며, 실제 여러 모니터링 플러그인들이 이런 아키테쳐들을 사용)
     + Node Logging Agent 구조
       * 노드 레벨 로깅을 기반으로 각 노드에 에이전트를 두는 방식으로, 각 노드에 DaemonSet으로 에이전트를 둬서 Worker Node에 만들어지는 이 경로으 ㅣ로그들을 조회하는 형태
       * 또는 Dokcer 로깅 드라이브를 수정해 바로 에이전트로 로그를 쓰게 할 수 있음
       * 즉, 읽은 로그를 수집 서버로 전송하는 패턴
     + Sidecar Container Streaming
       * 한 컨테이너에서 access.log와 app.log라는 두 종류의 로그 파일이 쌓인다고 가정을 하면, 이 두 로그 파일을 stdout하여 Worker Node에 파일이 쌓일 때, Log도 한 파일에 쌓임
       * 이럴 때, 메인 컨테이너로 Log를 studout으로 쓰는 것이 아닌 파일로만 저장하며, 별도의 컨테이너를 하나 더 만들어 해당 컨테이너 역할을 access.log 파일을 읽어서 stdout 출력을 하도록 함
         * 그럼 로그 파일 이름이 컨테이너 단위로 저장되므로 pod_access_숫자 이름으로 로그 파일이 생성
       * app.log도 마찬가지로 별도의 컨테이너를 만들어 stdout으로 출력하면, 로그 파일이 부리되는 구성
       * Logging Agent를 Sidecar 형태로 넣는 것으로, App 컨테이너는 stdout으로 출력하지 않아도 Agent 컨테이너에 의해 로그가 수집이 되고, 그 로그는 수집 서버로 바로 보내주는 구조

<div align="center">
<img src="https://github.com/user-attachments/assets/09ad34ca-eb00-4749-a0fd-1b79c642257f">
</div>

3. 대표적인 오픈소스 : Elastic에서 제공하는 솔루션
   - UI를 담당하는 Kibana와 조회와 분석으로 유명한 Elastic Search, Agent 로그로 Logstash라는 것을 좋아해 ELK Stack이라고 부름
   - 수집 에이전트로 유명한 에이전트인 Fluentd가 있고, Kubernetes 용으로 가볍게 나온 솔루션이 Fluent-bit
   - Logstash보다 Flent-bit를 Elastic Search와 연결해서 주로 사용하는 것을 EFK Stack이라 부름
   - 대표적으로 모니터링 시스템으로 유명한 것은 Prometheus : 자체 대시보드보다 Grafana라는 시각화 UI가 유명하므로 연결을 해서 주로 사용
   - Loki라고 하는 PLG Stack이라 불리는 모니터링도 존재
     + 마스터 노드와 워커 노드는 Pod 별로 로그 파일이 적재
     + PLG Stack을 설치하면 DaemonSet으로 Promtail에서 이 서비스로 로그 파이을 Push하며, delployment로 Grafana가 설치가 되고 생기는 30000번 포트로 접근을 해서 UI로 로그를 조회하게 되면, Granfana에서 Loki로 API 쿼리를 전송하면서 원하는 로그 조회 가능
