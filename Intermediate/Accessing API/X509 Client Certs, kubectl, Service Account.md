-----
### API 서버로 접근하는 3가지 방법 - X509 Client Certs, kubectl, Service Account
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/654eb5fd-939e-4395-b63a-557aa3275828">
</div>

1. Cluster에 6443 Port로 API 서버가 열려 있으며, 사용자는 HTTPS 접근을 하려면 Kuberentes 설치 시 kubeconfig라고 해서 Cluster에 접근할 수 있는 정보들이 있는데, 해당 파일 안에 인증서 내용이 존재 : 클라이언트 키와 인증서를 복사해서 가지고 와야 함
   - 최초 발급기관(CA)과 클라이언트(Client)에 대한 개인키를 만들고, 이 개인키를 가지고 인증서(CA csr, Client csr)를 만들기 위한 인증요청서라는 CSR 파일을 생성
   - CA의 경우, 인증요청서를 가지고 바로 인증서를 만듬 : kubeconfig에 있는 CA crt
   - 하지만, 클라이언트 인증서 경우 발급기간, 개인키와 인증서 그리고 클라이언트 요청 시 모두 합쳐서 만들어져서 클라이언트 인증서가 kubeconfig 파일에 Client crt로 Kubernetes API에 인증이 되서 리소스 조회 가능
   - accept-hosts라는 옵션을 통해 8001번 Port로 Proxy를 열어두면, 외부에서 HTTP로 접근할 수 있게 되는데, kubectl이 인증서를 가지고 있으므로 사용자는 아무런 인증서 없이 접근 가능

2. 외부 서버에 kubectl를 설치해서 Multi-Cluster에 접근
   - 사전에 각 클러스터에 있는 kubeconfig 파일이 사용자의 kubectl에도 존재해야 함
   - 사용자는 원하는 클러스터에 접근해 자원을 조회하고 생성 가능

3. kubeconfig
   - clusters 항목 : 클러스터를 등록할 수 있고, 이름과 연결 정보 CA 인증서가 존재
   - users 항목 : 사용자를 등록할 수 있는데, 유저 이름과 이 유저에 대한 개인키와 인증서가 존재
   - 따라서, 이를 context 항목을 통해서 이 둘을 연결할 수 있는데, 내용으로는 context, cluster, user
   - 이러한 과정을 거치면 Cluster A에 대한 Context가 생성되고, Cluster B에 대해서도 등록 가능
   - kubeconfig가 생성되면, 사용자가 클러스터 A에 연결하고 싶으면 kubectl config 명령으로 현재 사용하고 싶은 Context 지정 가능 및 지정했을 때 kubectl get node 명령을 통해 Cluster에 대한 Node 정보 조회 가능

4. Service Account
   - Secret이 존재 : 내용으로는 CA crt 정보와 토큰 값이 존재
   - Pod를 만들면, Service Account가 연결이 되고 Pod는 이 토큰값을 통해 API 서버에 연결할 수 있는데, 즉, 토큰값을 알면 사용자도 이 값을 가지고 API 서버에 접근할 수 있으
  
