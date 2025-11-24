-----
### Kubernetes Cluster 설치 - Windows
-----
1. 설치 스펙 : Windows10 / 11, Cpu 6core 이상, Memory 16GB 이상, 인터넷 사용 가능 환경
2. 설치 가이드
   - XShell 설치 : 생성될 Master/Woker Node에 접속할 툴 (기존에 쓰고 있는게 있으면 생략가능)
   - VirtualBox 설치 : VM 및 내부 네트워크 생성 툴
   - Vagrant 설치 및 k8s 설치 스크립트 실행 : 자동으로 VirtualBox를 이용해 VM들을 생성하고, K8S관련 설치 파일들이 실행됨
   - Worker Node 연결 : Worker Node들을 Master에 연결하여 쿠버네티스 클러스터 구축
   - 설치 확인 : Node와 Pod 상태 조회
   - 대시보드 접근 : Host OS에서 웹 브라우저를 이용해 클러스터 Dashboard에 접근
<div align="center">
<img src="https://github.com/user-attachments/assets/eac41822-d3d9-4736-9f2d-c10db786103d">
</div>

3. XShell 설치
   - 다운로드 URL : ```https://www.netsarang.com/en/free-for-home-school/```
   - 설치 후 (세션 - 세션 만들기) k8s-master(192.168.56.30:22), k8s-worker1(192.168.56.31:22), k8s-worker2(192.168.56.32:22) IP 등록

4. Virtualbox 설치 (7.1.6 버전) ​
  - Download : ```https://download.virtualbox.org/virtualbox/7.1.6/VirtualBox-7.1.6-167084-Win.exe```
  - Site : ```https://www.virtualbox.org/wiki/Downloads```
  - FAQ : microsoft visual C++ 관련 에러 해결방법

​5. Vagrant(2.4.3 버전) 설치 및 k8s 설치 스크립트 실행
  - Vagrant 설치
    + Win 버전 : ```https://releases.hashicorp.com/vagrant/2.4.3/vagrant_2.4.3_windows_amd64.msi```
    + Download site : ```https://developer.hashicorp.com/vagrant/downloads?product_intent=vagrant```
    + FAQ : Vagrant 관련 에러 해결방법 ```https://cafe.naver.com/kubeops/26```

  - Vagrant 명령 실행
    + 윈도우에서 cmd 실행
    +k8s 폴더 생성 및 이동
    +Vagrantfile 파일 다운로드
```bash
// 폴더 생성
C:\Users\사용자>mkdir k8s
C:\Users\사용자>cd k8s 

// Vagrant 스크립트 다운로드
C:\Users\사용자\k8s> curl -O https://raw.githubusercontent.com/k8s-1pro/install/main/under-thesea/k8s-cluster-1.27/vagrant-2.4.3/Vagrantfile

// Rocky Linux Repo 세팅
C:\Users\사용자\k8s> curl -O https://raw.githubusercontent.com/k8s-1pro/install/main/under-thesea/k8s-cluster-1.27/vagrant-2.4.3/rockylinux-repo.json
C:\Users\사용자\k8s> vagrant box add rockylinux-repo.json

// Vagrant Disk 설정 Plugin 설치 
C:\Users\사용자\k8s> vagrant plugin install vagrant-vbguest vagrant-disksize
```
  - vagrant 실행 (5 ~ 10분 소요)
```bash
C:\Users\사용자\k8s> vagrant up
```

6. vagrant 명령어 참고
  - vagrant up : VM 생성 및 스크립트 설치 (최초 VM생성 할때만 사용하며, 생성 이후 부터 VM 기동 / 중지는 Virtualbox UI를 사용하는걸 권장)
  - vagrant destroy : 가상머신 삭제 (vagrant up으로 VM 생성 중 에러가 났을 때 이 명령으로 삭제)

​7. Worker Node 연결
   - XShell을 통해 master 접속 (id/pw : root/vagrant)
   - kubeadm으로 token 발급 및 복사
```bash
[root@k8s-master ~]# kubeadm token create --print-join-command
kubeadm join 192.168.56.30:6443 --token bver73.wda72kx4afiuhspo --discovery-token-ca-cert-hash sha256:7205b3fd6030e47b74aa11451221ff3c77daa0305aad0bc4a2d3196e69eb42b7
```
   - join.sh 파일은 생성되지 않고 직접 위 kubeadm token 명령으로 토큰을 생성
   - worker node1 접속 후 토큰 붙여놓기 (id/pw : root/vagrant)
```bash
[root@k8s-node1 ~]# kubeadm join 192.168.56.30:6443 --token bver73.wda72kx4afiuhspo --discovery-token-ca-cert-hash sha256:7205b3fd6030e47b74aa11451221ff3c77daa0305aad0bc4a2d3196e69eb42b7
```

   - worker node2 접속 후 토큰 붙여놓기 반복
```bash
[root@k8s-node2 ~]# kubeadm join 192.168.56.30:6443 --token bver73.wda72kx4afiuhspo --discovery-token-ca-cert-hash sha256:7205b3fd6030e47b74aa11451221ff3c77daa0305aad0bc4a2d3196e69eb42b7
```
   - 설치 확인
     + XShell을 통해 master 접속 (id/pw = root/vagrant)
     + kubectl 명령어
```bash
[root@k8s-master ~]# kubectl get pod -A
[root@k8s-master ~]# kubectl get nodes
```

   - 대시보드 접근
```bash
https://192.168.56.30:30000/#/login
```
<div align="center">
<img src="https://github.com/user-attachments/assets/a7b04bef-998c-4b79-87d2-79aca5c6279d">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/1e5a47d4-9137-4baf-bdb0-1ebde056a099">
</div>

8. 참고 : Version 정보
<div align="center">
<img src="https://github.com/user-attachments/assets/a852727f-c084-4a67-b105-bbf951975fd2">
</div>


9. 네트워크 구성도
<div align="center">
<img src="https://github.com/user-attachments/assets/046b4ee6-9cb2-4bb9-8e89-a4e6c95c105e">
</div>
