-----
### Service (Discovery, Publishing, Registry, Load Balancing)
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/ec4a2c8a-db45-4c4c-8767-701bf4117291" />
</div>

1. Pod와 Selector가 selector와 labels로 연결
2. Service Publishing : 외부에서 Pod로 트래픽을 연결해주는 기능
   - type : NodeProt
     + Node Object가 존재하며, Port가 생성되면서, 외부에서 보낸 트래픽을 Kubernetes 내부 Pod로 전달 가능
    
   - Pod의 Container를 보면, App 기동 후 호출할 수 있는 API와 Port가 노출
     + targetPort는 Cotainer Port를 가리킴
     + nodePort를 설정하면 해당 Port 번호로 외부 Port가 생성

   - type을 넣지 않으면 ClusterIP (Kubernetes 내부 Pod만 접근하는 용도로 서비스를 생성할 때 사용)
   - ports의 port (필수 속성)

2. Service Discovery : Kubernetes가 내부 DNS를 이용해 서비스 이름을 API로 호출할 수 있게 해주는 기능
   - 서비스 이름과 Service Port를 넣어 API를 호출하면, 내부적으로 targetPort로 Forwarding되어 트래픽이 Pod로 전달
   - Pod와 Service는 생성 시 부여되는 IP가 존재하지만, Pod의 경우 삭제 시에 IP가 변경되므로 항상 서비스를 통해 호출해야 함
   - Kubernetes는 서비스 이름을 자동으로 내부 DNS로 등록해주므로 도메인명으로 쉽게 원하는 Pod의 API 전송 가능
   - 💡 Cluster의 다른 Namespace에 있는 Pod가 호출될 때는, 서비스 이름 뒤 해당 Pod가 속해있는 네임스페이스 이름까지 넣어줘야 함

3. Service와 Pod 설정에서 자주 사용하는 패턴
   - App의 Port가 변경되면, Service의 targetPort도 변경되는데, Container의 Port가 변경되더라도 서비스까지 신경쓰지 않는 방법
   - Pod에도 containers/ports라는 속성 존재
     + name : Port의 성격
     + containerPort : 실제 App이 노출되고 있는 Port (정보성 속성)
     + Service의 targetPort에 http와 같이 Name으로 지정 가능 : Pod에서 name Mapping을 통해 Port를 찾아 API 전송
    
4. Service Registry (Deployment Update)
   - Pod IP 관리 :업데이트 과정 속 Pod가 삭제되고, 새로 만들어지는데 Kubernetes가 Service에 호출되는 IP를 제거하고 등록해주는 것을 바로 해주므로, 신경쓸 필요가 없음
   
5. Load Balancing
   - 서비스에 트래픽을 날리면 알아서 두 Pod로 트래픽을 분배도 해줌

6. 동작 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/99268dbd-78d6-4b5c-88c6-51db21fe0c98" />
</div>

  - Pod 내부에서 Service 명으로 API 호출 [서비스 디스커버리] 
```bash
// Version API 호출
curl http://api-tester-1231:80/version
```

  - Deployment에서 Pod의 ports 전체 삭제, Service targetPort를 http에서 8080으로 수정
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: anotherclass-123
  name: api-tester-1231
spec:
  template:
    spec:
      nodeSelector:
        kubernetes.io/hostname: k8s-master
      containers:
        - name: api-tester-1231
          ports:                // 삭제
          - name: http          // 삭제
            containerPort: 8080 // 삭제
---
apiVersion: v1
kind: Service
metadata:
  namespace: anotherclass-123
  name: api-tester-1231
spec:
  ports:
    - port: 80
      targetPort: http -> 8080 // 변경
      nodePort: 31231
  type: NodePort
```

  - 다시 Pod 내부에서 Service 명으로 API 호출
```bash
curl http://api-tester-1231:80/version
```
