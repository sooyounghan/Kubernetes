-----
### Authentication - X509 Certs, Kubectl, ServiceAccount
-----
<img width="886" height="232" alt="image" src="https://github.com/user-attachments/assets/e37f6811-cb8b-470c-b489-66e7f7360367" />

1. X509 Client Certs
   - 외부에서 Cluster로 API를 보내는 방법으로는 Client key와 crt파일을 이용해서 API Server에 https 로 접근
   - 그리고 Cluster 관리자가 kubectl을 이용해서 내부 서버에 Proxy를 만들고 http로 접근
   - kubeconfig 인증서 확인 (base64로 변환)
```bash
cat /etc/kubernetes/admin.conf

// client.crt 내용 확인
grep 'client-certificate-data' /etc/kubernetes/admin.conf | head -n 1 | awk '{print $2}' | base64 -d

// client.key 내용 확인
grep 'client-key-data' /etc/kubernetes/admin.conf | head -n 1 | awk '{print $2}' | base64 -d
```
   - Windows에서는 PFX (PKCS#12) 형식을 더 잘 지원하므로, 인증서와 키를 묶어서 .pfx 파일을 생성
```bash
grep 'client-certificate-data' /etc/kubernetes/admin.conf | head -n 1 | awk '{print $2}' | base64 -d > client.crt
grep 'client-key-data' /etc/kubernetes/admin.conf | head -n 1 | awk '{print $2}' | base64 -d > client.key
openssl pkcs12 -export -out client.pfx -inkey client.key -in client.crt
Enter Export Password: (엔터)
Verifying - Enter Export Password: (엔터)
```
   - 내 컴퓨터로 파일 이동 (FTP tool 이용 필요) [XFTP 다운로드 : ```https://www.netsarang.com/en/free-for-home-school/```]
<img width="897" height="476" alt="image" src="https://github.com/user-attachments/assets/6f536b9c-1c62-499d-aa3a-7247db4c5b84" />
```bash
curl --cert client.pfx -k https://192.168.56.30:6443/api/v1/nodes
```

  - CMD에서 Https API 호출 : 디릭토리는 ```C:\Users\<사용자>```를 기준 (```C:\Users\<사용자>\client.pfx```)
```bash
// curl 이용시
C:\Users\taemin>curl --cert client.pfx -k https://192.168.56.30:6443/api/v1/nodes

// postman 이용시
https://192.168.56.30:6443/api/v1/nodes
Settings > General > SSL certificate verification > OFF
Settings > Certificates > Client Certificates > Host(192.168.56.30:6443), CRT file(client.crt), KEY file(client.key)
```

   - Master 내부의 kubectl로 proxy를 띄어서 외부에서는 http로 호출 하기
   - Cluster 설치시 Master에서 실행된 스크립트 내용 (아래 내용을 실행할 필요 없음)
```bash
// kubeadm / kubectl / kubelet 설치
yum install -y --disableexcludes=kubernetes kubelet-1.27.2-0.x86_64 kubeadm-1.27.2-0.x86_64 kubectl-1.27.2-0.x86_64

// admin.conf 인증서 복사
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
   - Master Node에서 kubectl로 proxy 띄우기 (^*$ : 어느 Host에서나 접근 가능)
```bash
nohup kubectl proxy --port=8001 --address=192.168.56.30 --accept-hosts='^*$' >/dev/null 2>&1 &
```

   - PC의 CMD에서 HTTP로 API 호출
```bash
// curl
curl http://192.168.56.30:8001/api/v1/nodes

// postman
http://192.168.56.30:8001/api/v1/nodes
```

2. kubectl
   - 클러스터(cluster-B)를 하나 더 설치
   - kubectl의 config를 기능을 이용하면 여러 클러스터가 있어도 원하는 클러스터를 선택해서 cli 호출 가능
   - 새 클러스터(cluster-b) 구축 
```bash
// 폴더 생성
C:\Users\사용자>mkdir k8s2
C:\Users\사용자>cd k8s2

// Vagrant 스크립트 다운로드
curl -O https://raw.githubusercontent.com/k8s-1pro/install/main/ground/k8s-1.27/vagrant-2.4.3-slim/Vagrantfile
vagrant up
```
   - Cluster A에서 kubeconfig 내용 확인
```bash
cat /etc/kubernetes/admin.conf
```
   - Cluster B에서 kubeconfig 내용 확인
```bash
cat /etc/kubernetes/admin.conf
```

  - 아래와 같이 kubeconfig 내용 병합하여 config 파일 생성
    + config 파일 위치 : ```C:\Users\<사용자>\.kube\config```
```bash
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1KVEUtLS0tLQo=
    server: https://192.168.0.30:6443
  name: cluster-a
- cluster:
    certificate-authority-data: LS0tLS1KVEUtLS0tLQo=
    server: https://192.168.0.50:6443
  name: cluster-b
contexts:
- context:
    cluster: cluster-a
    user: admin-a
  name: context-a
- context:
    cluster: cluster-b
    user: admin-b
  name: context-b
current-context: context-a
kind: Config
preferences: {}
users:
- name: admin-a
  user:
    client-certificate-data: LS0tLS1KVEUtLS0tLQo=
    client-key-data: LS0tLS1KVEUtLS0tLQo=
- name: admin-b
  user:
    client-certificate-data: LS0tLS1KVEUtLS0tLQo=
    client-key-data: LS0tLS1KVEUtLS0tLQo=
```

   - PC에서 kubectl 다운로드 후 클러스터 별 cli 호출
```bash
C:\Users\사용자> curl.exe -LO "https://dl.k8s.io/release/v1.27.0/bin/windows/amd64/kubectl.exe"
kubectl config use-context context-a
kubectl get nodes
kubectl config use-context context-b
kubectl get nodes
```

   - 이후 새 클러스터 삭제
```bash
C:\Users\사용자\k8s2> vagrant destroy
```

3. Service Account
   - ServiceAccount를 이용하면 연결된 Secret의 token을 이용해, 특정 Pod만 선택적으로 API 호출가능
   - Kubernetes v1.24부터 ServiceAccount Token 자동 생성 기능이 비활성화됨
   - Namespace 
```bash
kubectl create ns nm-01
```
   - ServiceAccount 및 Secret 확인 : Kubernetes v1.24부터 ServiceAccount에 Secret Token 생성 기능이 비활성화 되서 직접 추가 필요
```bash
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Secret
metadata:
  name: nm-01
  namespace: nm-01
  annotations:
    kubernetes.io/service-account.name: default
type: kubernetes.io/service-account-token
EOF
```
   - 확인
```bash
kubectl describe -n nm-01 serviceaccounts
kubectl describe -n nm-01 secrets
```
```bash
[root@k8s-master ~]# kubectl describe -n nm-01 serviceaccounts
Name:                default
Namespace:           nm-01
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   <none>
Tokens:              nm-01
Events:              <none>

[root@k8s-master ~]# kubectl describe -n nm-01 secrets
Name:         nm-01
Namespace:    nm-01
Labels:       <none>
...
```

   - Pod 생성
```bash
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: nm-01
  labels:
     app: pod
spec:
  containers:
  - name: container
    image: kubetm/app
EOF
```
   - PC의 CMD에서 Token을 이용해 API 호출
```bash
// curl
curl -k https://192.168.56.30:6443/api/v1 -H "Authorization: Bearer <token>"
curl -k https://192.168.56.30:6443/api/v1/namespaces/nm-01/pods/ -H "Authorization: Bearer <token>"

// postman
# [header] Authorization : Bearer TOKEN
https://192.168.56.30:6443/api/v1
https://192.168.56.30:6443/api/v1/namespaces/nm-01/pods/
```
  - 생성된 리소스는 추후 사용해야 하므로 삭제하지 않음
