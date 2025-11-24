-----
### Kubernetes 시작하기
-----
1. 시나리오
<div align="center">
<img src="https://github.com/user-attachments/assets/1c8b6d0e-7710-435e-bffa-8a0000a4e536">
</div>

2. 최초 Linux 서버에서 Hello World라는 Node.js 앱을 하나 생성해 적재
   - 해당 Linux에는 Node.js를 실행할 수 있는 패키지가 설치되어 있어서 앱이 구동될 것

3. Dokcer가 설치된 다른 서버로 이동하여 앞에서 만든 Hello 앱을 그대로 가져올 것
   - 하지만 Node.js가 설치되어 있지 않으므로, 실행이 되지 않을 것
   - Docker를 이용해 Container를 생성 (Dokcer Hub(여러 컨테이너 이미지를 공개적으로 올림)를 이용해서 이미지를 가져옴
   - 해당 이미지를 통해 하나의 컨테이너를 생성

4. 컨테이너가 만들어지면, Docker로 이 컨테이너를 구동시켜서 앱에서 서비스를 할 수 있도록 Open
5. 다음으로, Kubernetes를 사용해 앱을 띄울 것
   - 앞에서 만든 Container Image를 다시 Docker Hub에 적재
   - Pod와 Pod 안에 컨테이너를 생성할 때, 방금 올린 Docker Hub에서 이미지를 가져와서 Pod를 구동
   - Pod (생성할 컨테이너 묶음)과 Service(외부에서 접근할 수 있는 서비스) 생성
