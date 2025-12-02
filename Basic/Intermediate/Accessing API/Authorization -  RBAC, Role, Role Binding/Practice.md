-----
### Authorization - RBAC, Role, RoleBinding
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/4dbbac02-0879-4f3a-b92a-13195cd9506f">
</div>

1. Role, RoleBinding
   - Namespace(nm-01), Pod(pod-1), ServiceAccount(default), Secret(nm-01) 그대로 사용
   - Role
     + API Method 참고 자료 : ```https://kubernetes.io/docs/reference/access-authn-authz/authorization/#review-your-request-attributes```
```yaml
apiVersion: rbac.authorization.k8s.io/v1 # Core API가 아님
kind: Role
metadata:
  name: r-01
  namespace: nm-01
rules:
- apiGroups: [""] # 별도 그룹을 지정하지 않아도 됨
  verbs: ["get", "list"] # API의 Method 지정
  resources: ["pods"] # Core API
```
  - RoleBinding 
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rb-01
  namespace: nm-01
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: r-01
subjects:
- kind: ServiceAccount
  name: default
  namespace: nm-01
```
  - Service 
```yaml
apiVersion: v1 # Core API이므로 별도 그룹을 넣지 않아도 됨
kind: Service
metadata:
  name: svc-1
  namespace: nm-01
spec:
  selector:
    app: pod
  ports:
  - port: 8080
    targetPort: 8080
```
   - PC의 CMD에서 Token을 이용해 API 호출 ​(Role에 따라 Pod는 조회되나, Service는 조회 안됨)
```bash
// curl로 Pod List 조회
curl -k https://192.168.56.30:6443/api/v1/namespaces/nm-01/pods -H "Authorization: Bearer <token>" 

// curl로 Service List 조회 (조회 불가)
curl -k https://192.168.56.30:6443/api/v1/namespaces/nm-01/services-H "Authorization: Bearer <token>" 

// postman
# [header] Authorization : Bearer TOKEN
https://192.168.56.30:6443/api/v1/namespaces/nm-01/pods/
```

2. ClusterRole, ClusterRoleBinding
   - Namespace 
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-02
```
   - ServiceAccount 및 Secret 생성 
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sa-02
  namespace: nm-02
---
apiVersion: v1
kind: Secret
metadata:
  namespace: nm-02
  name: sa-02-token
  annotations:
    kubernetes.io/service-account.name: sa-02
type: kubernetes.io/service-account-token
```
   - ClusterRole
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cr-02
rules:
- apiGroups: ["*"]
  verbs: ["*"]
  resources: ["*"]
```
  - ClusterRoleBinding 
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: rb-02
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cr-02
subjects:
- kind: ServiceAccount
  name: sa-02 
  namespace: nm-02
```
   - PC의 CMD에서 Token을 이용해 API 호출 (ClusterRole에 따라 Pod와 Service 모두 조회 가능)
```bash
curl -k https://192.168.56.30:6443/api/v1/namespaces/nm-01/pods -H "Authorization: Bearer <token>" 
curl -k https://192.168.56.30:6443/api/v1/namespaces/nm-01/services -H "Authorization: Bearer <token>" 

// postman
# [header] Authorization : Bearer TOKEN
https://192.168.56.30:6443/api/v1/namespaces/nm-01/pods/
https://192.168.56.30:6443/api/v1/namespaces/nm-01/service/
```

   - 실습 후 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 or Master Node에서 아래 명령 실행)
```bash
kubectl delete clusterrole cr-02
kubectl delete clusterrolebinding rb-02
kubectl delete ns nm-01 nm-02
```
