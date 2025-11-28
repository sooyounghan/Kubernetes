-----
### Kubernetes Architecture
------
<div align="center">
<img src="https://github.com/user-attachments/assets/a6ba0605-ce06-442d-af8d-7b09aa86444c">
</div>

1. Kubernetes는 한 대의 Master와 여러 대의 Work Node들로 구성
   - 마스터에는 Control Plane Component라고 하는 Kubernetes 주요 기능을 담당하는 컴포넌트들이 존재
   - 각각의 Worker Node에는 Worker Componen라고 하는 컨테이너를 관리하기 위한 기능이 존재

2. Kubernetes는 크게 서비스와 Pod에 대한 네트워크 영역이 존재
   - Service Network : 서비스를 Pod에 붙였을 때 이 서비스를 통해 Pod로 통신 가능
   - Pod Network : Calico Network Plugin으로 Pod 간 통신

3. Storage : 데이터를 안정적으로 저장하는 방식
   - hostPath를 사용하는 방법
   - Storage를 사용하는 방법
   - 내부 클러스터 내에 Third-Party에서 제공하는 Storage Solution을 설치하는 방법이 존재
   - 어떤 방식의 Storage를 사용하던 간에 Volume의 Type에는 3가지 종류로 나눠질 수 있음 : FileStorage, BlockStorage, ObjectStorage

4. Logging : Kubernetes 내에 돌아가고 있는 App에 떨어지는 로그들을 어떻게 관리하느냐에 대한 부분
   - Service Pipeline : 별도의 플러그인을 설치하게 되면 모니터링에 관련된 Pod들이 생기고, 이 Pod들이 각각의 노드에서 로그를 가져다가 수집 서버에 모으고 UI를 통해 사용자에게 보여주는 방식
   - Core Pipeline : Pod에서 생성된 로그가 어떤 구조로 쌓여있는지 그리고 쌓여진 로그들을 어떻게 보는지에 대한 부분
