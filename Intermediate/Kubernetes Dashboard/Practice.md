-----
### Dashaboard - Kubeconfig, Token
-----
1. Role, RoleBinding
   - ClusterRole, ClusterRoleBinding 확인
```bash
kubectl describe clusterrole cluster-admin
kubectl describe clusterrolebindings kubernetes-dashboard2
```
```bash
[root@k8s-master ~]# kubectl describe clusterrole cluster-admin
Name:         cluster-admin
Labels:       kubernetes.io/bootstrapping=rbac-defaults
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  *.*        []                 []              [*]
             [*]                []              [*]

[root@k8s-master ~]# kubectl describe clusterrolebindings kubernetes-dashboard2
Name:         kubernetes-dashboard2
Labels:       k8s-app=kubernetes-dashboard
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  cluster-admin
Subjects:
  Kind            Name                  Namespace
  ----            ----                  ---------
  ServiceAccount  kubernetes-dashboard  kubernetes-dashboard
```

   - Token 생성
```yaml
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Secret
metadata:
  name: kubernetes-dashboard-token
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/service-account.name: "kubernetes-dashboard"   
type: kubernetes.io/service-account-token  
EOF
```
   - Token 확인
```bash
kubectl -n kubernetes-dashboard get secret kubernetes-dashboard-token -o jsonpath='{.data.token}' | base64 --decode
````
```bash
[root@k8s-master ~]# kubectl -n kubernetes-dashboard get secret kubernetes-dashboard-token -o jsonpath='{.data.token}' | base64 --decode
ey...
```

  - ServiceAccount에 Token 확인
```bash
kubectl describe -n kubernetes-dashboard sa kubernetes-dashboard
```
```bash
[root@k8s-master ~]# kubectl describe -n kubernetes-dashboard sa kubernetes-dashboard
Name:                kubernetes-dashboard
Namespace:           kubernetes-dashboard
Labels:              k8s-app=kubernetes-dashboard
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   <none>
Tokens:              kubernetes-dashboard-token
Events:              <none>
```

  - Master에서 client.cer 파일 생성 후 내 PC로 복사해서 인증서 설치
```bash
grep 'client-certificate-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> client.crt
grep 'client-key-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> client.key
openssl pkcs12 -export -clcerts -inkey client.key -in client.crt -out client.p12 -name "k8s-master-30"
Enter Export Password: (엔터)
Verifying - Enter Export Password: (엔터)

openssl pkcs12 -in client.p12 -clcerts -nokeys -out client.cer
Enter Import Password: (엔터)
```

   - Chrome 확장 프로그램 (Header에 Token을 넣어야 대시보드 화면이 열리는 데, 브라우저에 Token을 넣을 수 있도록 해주는 프로그램)
     + ModHeader : ```https://chromewebstore.google.com/detail/modheader-modify-http-hea/idgpnmonknjnojddfkpgkljpfnnfcklj?pli=1```
     + 확장 프로그램 오픈 후 Header 정보 입력
       * Authorization : Bearer <token> (Bearer 한칸 띄고 토큰 입력)
       * Token 확인 명령
```yaml
kubectl -n kubernetes-dashboard get secret kubernetes-dashboard-token -o jsonpath='{.data.token}' | base64 --decode
```
<div align="center">
<img src="https://github.com/user-attachments/assets/bab91c78-1e8b-470f-90e3-111586933f76">
</div>

  - URL 입력
```
https://192.168.56.30:6443/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login
```

   - Dashboard에서 토큰 입력(Beare 없이 token만) 후 로그인 
<div align="center">
<img src="https://github.com/user-attachments/assets/f56fb116-9e90-422a-a7db-1cf92bee0ccd">
</div>
