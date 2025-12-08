-----
### 손쉽게 데브옵스 환경을 구축하는 방법
-----
1. Vagrant 스크립트 실행
  - 윈도우 > 실행 > cmd > 확인
```bash
# Vagrant 폴더 생성
C:\Users\사용자> mkdir cicd
C:\Users\사용자> cd cicd
```
```bash
# Vagrant 스크립트 다운로드
C:\Users\사용자\cicd> 
curl -O https://raw.githubusercontent.com/k8s-1pro/install/main/ground/cicd-server/vagrant-2.4.3/Vagrantfile
```
```bash
# Rocky Linux Repo 세팅
curl -O https://raw.githubusercontent.com/k8s-1pro/install/main/ground/cicd-server/vagrant-2.4.3/rockylinux-repo.json
vagrant box add rockylinux-repo.json
```
```bash
# Vagrant Vbguest 및 Disk Plugin 설치 
vagrant plugin install vagrant-vbguest vagrant-disksize
```
```bash
# Vagrant 실행
vagrant up
```

2. CI/CD 서버로 접속 (Windows)
  - Sessions > New session을 선택해서 접속 세션 생성
  - 최초 id는 root, password는 vagrant 
  - 참고 이미지
<div align="center">
<img src="https://github.com/user-attachments/assets/7227f61f-b111-44fa-828b-6066ca8358f4" />
</div>

3. Jenkins 초기 세팅
  - CI/CD 서버에서 초기 비밀번호 확인
```bash
[root@cicd-server ~]# cat /var/lib/jenkins/secrets/initialAdminPassword
```

  - Jenkins 대시보드 접속해서 확인한 비밀번호 입력
```bash
http://192.168.56.20:8080/login
```
<div align="center">
<img src="https://github.com/user-attachments/assets/a601c1ac-88fc-4ea8-bbb2-3e1064251bdb" />
</div>

  - 플러그인 설치
<div align="center">
<img src="https://github.com/user-attachments/assets/00188245-2b51-4bfb-a252-d403bc81a094" />
</div>

  - Admin 사용자 생성
<div align="center">
<img src="https://github.com/user-attachments/assets/381a145c-cf36-48a4-b558-de02ac0f1dc1" />
</div>

   - [Save and Finish]  -> [Start using Jenkins]

4. 전역 설정 (JDK, Gradle)
   - [Dashboard] > 화면 상단 [Jenkins 관리] 버튼 클릭
<div align="center">
<img src="https://github.com/user-attachments/assets/6723a044-ad28-4d41-b064-799cefc269bc" />
</div>

   - JDK 세팅 : 본인의 JDK17 설치 경로 확인 (설치시마다 마이너 버전들이 업데이트 되기 때문에 개인마다 경로가 달라짐)
```bash
[root@cicd-server ~]# find / -name java | grep java-17-openjdk
/usr/lib/jvm/java-17-openjdk-17.0.7.0.7-3.el8.x86_64/bin/java  // -->> /bin/java 제외하고 복사
```
   - 꼭 /bin/java 부분은 제외하고 복사
   - 위 결과 복사해서 아래 JAVA_HOME에 넣기
```bash
# Name : jdk-17
# JAVA_HOME : /usr/lib/jvm/java-17-openjdk-17.0.7.0.7-3.el8.x86_64
```
<div align="center">
<img src="https://github.com/user-attachments/assets/cc31ff52-8826-4ab7-9e25-8aee42e82f52" />
</div>

  - /usr/lib/jvm/java-17-openjdk-17.0.7.0.7-3.el8.x86_64 doesn’t look like a JDK directory 메세지 무시
  - Gradle 세팅
```bash
# Name : gradle-7.6.1
# GRADLE_HOME : /opt/gradle/gradle-7.6.1
```
<div align="center">
<img src="https://github.com/user-attachments/assets/2d04b221-71f3-4bb5-9ac7-cd6f4e5bbbcd" />
</div>

   - Install automatically이 기본으로 체크
   - [체크해제]를 해야지 GRADLE_HOME 입력란이 나옴
   - [Save]

5. Docker Hub 가입
  - Docker 계정 생성
```bash
https://hub.docker.com/signup
```
<div align="center">
<img src="https://github.com/user-attachments/assets/2d4e311f-d018-48c1-adef-0376019d6dc6" />
</div>

   - 컨테이너 이미지 명으로 1pro/app-tester가 아닌 <Username>/app-tester 로 바꿔서 사용

6. Docker 사용 설정
```bash
# jeknins가 Docker를 사용할 수 있도록 권한 부여
[root@cicd-server ~]# chmod 666 /var/run/docker.sock
[root@cicd-server ~]# usermod -aG docker jenkins

# Jeknins로 사용자 변경 
[root@cicd-server ~]# su - jenkins -s /bin/bash

# 자신의 Dockerhub로 로그인 하기
[jenkins@cicd-server ~]$ docker login
Username: 
Password: 
```
   - 자신의 Docker Hub Username 및 Password 입력
   - Login Succeeded 결과 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/0bae8747-08c6-4fd9-8c50-89c299eaaccb" />
</div>

7. Master Node에서 인증서 복사
  - CI / CD 서버에서 SCP 명령으로 인증서 가져오기
  - 해당 작업을 하기전에 Master Node가 정상적으로 실행중인지 확인
  - root가 아닌 Jenkins 유저 상태에서 작업
```bash
# 폴더 생성
[jenkins@cicd-server ~]$ mkdir ~/.kube

# Master Node에서 인증서 가져오기
[jenkins@cicd-server ~]$ scp root@192.168.56.30:/root/.kube/config ~/.kube/
```

  - 인증서 가져오기 실행 후 [fingerprint] yes 와 [password] vagrant 입력
<div align="center">
<img src="https://github.com/user-attachments/assets/7bd9cb86-19c5-4046-80ac-5d5c5fdcbaa5" />
</div>

  - 동작 확인
```bash
# kubectl 명령어 사용 확인
[jenkins@cicd-server ~]$ kubectl get pods -A
```

8. GitHub 가입
  - GitHub 계정 생성
```
https://github.com/signup
```
<div align="center">
<img src="https://github.com/user-attachments/assets/74c76dce-ff4d-4590-968f-bd7446830943" />
</div>

   - ```https://github.com/k8s-1pro/kubernetes-anotherclass-sprint2``` -> ```https://github.com/<username>/kubernetes-anotherclass-sprint2```
<div align="center">
<img src="https://github.com/user-attachments/assets/eb3c628d-b179-4209-91dd-85ddf49aff82" />
</div>

   - Fork 생성
<div align="center">
<img src="https://github.com/user-attachments/assets/e0055e40-d2ef-416e-9a43-4029aa3c9dcf" />
</div>

  - Deployment Yaml 파일 수정
  - 2121 폴더 클릭
<div align="center">
<img src="https://github.com/user-attachments/assets/b4415662-1733-4331-8d37-3083a088183f" />
</div>

  - deployment.yaml 파일 수정
<div align="center">
<img src="https://github.com/user-attachments/assets/7f373448-34cf-4d3c-ba86-4bc98bdcb1cd" />
</div>


   - image의 값을 자신의 DockerHub Username 으로 수정하기
<div align="center">
<img src="https://github.com/user-attachments/assets/f72c7c1b-b812-461d-a8d4-fd151ac611db" />
</div>


   - 이따금씩 Sync fork 확인하기
<div align="center">
<img src="https://github.com/user-attachments/assets/8d7ff8dc-3e30-4eb5-9695-523b731f0c44" />
</div>
