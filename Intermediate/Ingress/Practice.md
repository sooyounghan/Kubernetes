-----
### Practice(Ingress - Service Loadbalancing, Canary Upgrade)
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/c68b6f9d-ccab-4064-a291-fbc8e616d255">
</div>

1. Nginx Controller
    - Nginx와 같은 Ingress Controller 컨트롤러를 별도로 설치하지 않으면, 아무리 쿠버네티스 내에 Ingress를 만들어도 동작하지 않음
    - Nginx는 쿠버네티스 내에 Ingress가 만들어지면 해당 Rule을 읽고, 그 Rule에 따라 트래픽을 분배해주는 역할
    - 그렇기 때문에 분배시키려는 트래픽은 반드시 Nginx로 보내야 함
    - Nginx 설치
```bash
kubectl apply -f https://raw.githubusercontent.com/k8s-1pro/install/refs/heads/main/ground/k8s-1.27/nginx-1.8.2/nginx-controller.yaml
```
   - Service(NodePort) 와 Port(30431, 30798) 설정은 위 배포 파일에 포함되어 있음

2. Service Loadbalancing
    - 외부 사용자(혹은 서버)가 보내는 API의 URL Path에 따라 다른 Pod로 트래픽 전송 가능
    - Shopping Page
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-shopping
  labels:
    category: shopping
spec:
  containers:
  - name: container
    image: kubetm/shopping
---
apiVersion: v1
kind: Service
metadata:
  name: svc-shopping
spec:
  selector:
    category: shopping
  ports:
  - port: 8080
```
   - Customer Center
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-customer
  labels:
    category: customer
spec:
  containers:
  - name: container
    image: kubetm/customer
---
apiVersion: v1
kind: Service
metadata:
  name: svc-customer
spec:
  selector:
    category: customer
  ports:
  - port: 8080
```
   - Order Service
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-order
  labels:
    category: order
spec:
  containers:
  - name: container
    image: kubetm/order
---
apiVersion: v1
kind: Service
metadata:
  name: svc-order
spec:
  selector:
    category: order
  ports:
  - port: 8080
```
   - 각 Pod 접근
```bash
[root@k8s-master ~]# curl 10.104.206.15:8080
Order Service.
[root@k8s-master ~]# curl 10.104.217.191:8080
Customer Service.
[root@k8s-master ~]# curl 10.110.100.61:8080
Shopping Service.
```

   - Ingress (버전(v1beta1 -> v1)이 변경되면서 Yaml 스펙이 변경)
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: service-loadbalancing
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: / # root
        pathType: Prefix
        backend:
          service:
            name: svc-shopping # svc-shopping으로 연결
            port:
              number: 8080
      - path: /customer
        pathType: Prefix
        backend:
          service:
            name: svc-customer
            port:
              number: 8080
      - path: /order
        pathType: Prefix
        backend:  
          service:
            name: svc-order
            port:
              number: 8080
```
   - Master에서 각 Service별 API 호출
```bash
curl 192.168.56.30:30431/
curl 192.168.56.30:30431/order
curl 192.168.56.30:30431/customer
```
```bash
[root@k8s-master ~]# curl 192.168.56.30:30431/
Shopping Service.
[root@k8s-master ~]# curl 192.168.56.30:30431/order
Order Service.
[root@k8s-master ~]# curl 192.168.56.30:30431/customer
Customer Service.
```

3. Canary Upgrade
    ​- Ingress에 annotation 설정을 통해 트래픽을 제어 할 수 있음
    - Nginx가 해당 설정을 읽어서 트래픽을 조절하는 역할
    - weight(10%) : 트래픽의 10%를 지정된 Pod로 보냄
    - header : API Header를 보고 트래픽을 지정된 Pod로 보냄
    - App V1 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-v1
  labels:
    app: v1
spec:
  containers:
  - name: container
    image: kubetm/app:v1
---
apiVersion: v1
kind: Service
metadata:
  name: svc-v1
spec:
  selector:
    app: v1
  ports:
  - port: 8080
```
   - App V2
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-v2
  labels:
    app: v2
spec:
  containers:
  - name: container
    image: kubetm/app:v2
---
apiVersion: v1
kind: Service
metadata:
  name: svc-v2
spec:
  selector:
    app: v2
  ports:
  - port: 8080
```
   - Ingress - default
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app
spec:
  ingressClassName: nginx
  rules:
  - host: www.app.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-v1
            port:
              number: 8080
```
   - Master에 Hostname 등록 후 도메인으로 API 호출
```bash
cat << EOF >> /etc/hosts
192.168.56.30 www.app.com
EOF

curl www.app.com:30431/version
```
```bash
[root@k8s-master ~]# curl www.app.com:30431/version
Version : v1
```

   - Ingress - weight
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: canary-v2
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"
spec:
  ingressClassName: nginx
  rules:
  - host: www.app.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-v2
            port:
              number: 8080
```

   - API 호출시 10% 확률로 v2 호출 확인
```bash
while true; do curl www.app.com:30431/version; sleep 1; done
```
```bash
[root@k8s-master ~]# while true; do curl www.app.com:30431/version; sleep 1; done
Version : v2
Version : v1
Version : v1
Version : v1
Version : v1
Version : v1
Version : v1
Version : v1
Version : v1
Version : v1
Version : v1
Version : v1
```

  - Ingress - header  : 기존 Ingress(canary-v2) 삭제 후 아래 Ingress 생성
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: canary-kr
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-by-header: "Accept-Language"
    nginx.ingress.kubernetes.io/canary-by-header-value: "kr"
spec:
  ingressClassName: nginx
  rules:
  - host: www.app.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-v2
            port:
              number: 8080
```
   - API 호출
```bash
curl -H "Accept-Language: kr" www.app.com:30431/version
```
```bash
[root@k8s-master ~]# curl -H "Accept-Language: kr" www.app.com:30431/version
Version : v2
```

4. Https
   - Ingress에 Secret으로 TLS를 연결할 수 있고, 그러면 해당 Pod에 연결하기 위해 외부에서는 https로 접근됨
   - App V1 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-https
  labels:
    app: https
spec:
  containers:
  - name: container
    image: kubetm/app
---
apiVersion: v1
kind: Service
metadata:
  name: svc-https
spec:
  selector:
    app: https
  ports:
  - port: 8080
```
   - Ingress - TLS
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: https
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - www.https.com
    secretName: secret-https
  rules:
  - host: www.https.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-https
            port:
              number: 8080
```
   - Secret
```bash
# 인증서 생성
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=www.https.com/O=www.https.com"

# Secret 생성
kubectl create secret tls secret-https --key tls.key --cert tls.crt
```

   - Windows HostName 등록
      + 파일 위치 : C:\Windows\System32\drivers\etc\hosts
```bash
192.168.56.30 www.https.com
```
   - 브라우저에서 접속
```bash
https://www.https.com:30798/hostname
```

   - 실습 후 모든 리소스 삭제 ​(Dashboard에서 리소스별 삭제 or Master Node에서 아래 명령 실행)
```bash
kubectl delete pod pod-shopping pod-customer pod-order pod-v1 pod-v2 pod-https
kubectl delete svc svc-shopping svc-customer svc-order svc-v1 svc-v2 svc-https
kubectl delete ingress service-loadbalancing app canary-v2 canary-kr https
```
