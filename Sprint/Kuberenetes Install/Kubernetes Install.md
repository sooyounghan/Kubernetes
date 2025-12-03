-----
### Kubeadm을 이용해 쉽고 빠르게 설치하는 방법
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/1d71f6a6-401a-4b7e-a909-2b9c3ac6e28b">
</div>

1. VirtualBox, Vargrant 설치 후, Vagrant 스크립트 실행 : 인프라 환경 설치
2. Kubernetes 작동을 위한 접속 툴을 설치해 Linux로 원격 접속
   - Kubernetes는 Pod 형태로 구성되어 있으므로, Pod 조회 명령을 통해 상태 확인

3. 상태가 정상이면 대시보드에 접속
4. 실습
   - 설치 권고 환경 : Windows10/11, Cpu 4core 이상, Memory 12GB 이상, 인터넷 사용 가능 환경
   - Virtualbox 설치 (7.1.6 버전)  (25.3.03 Version Update)
     + Download : ```https://download.virtualbox.org/virtualbox/7.1.6/VirtualBox-7.1.6-167084-Win.exe```
     + Site : ```https://www.virtualbox.org/wiki/Downloads```
     + FAQ : Microsoft visual C++ 관련 에러 해결방법 (```https://cafe.naver.com/kubeops/25```)

   - Vagrant 설치 (2.4.3 버전) (25.3.03 Version Update)
     + Download : ```https://releases.hashicorp.com/vagrant/2.4.3/vagrant_2.4.3_windows_amd64.msi```
     + Site : ```https://developer.hashicorp.com/vagrant/downloads?product_intent=vagrant```
     + Vagrant 스크립트 실행 : 윈도우 > 실행 > cmd > 확인
```bash
# Vagrant 폴더 생성
C:\Users\사용자> mkdir k8s && cd k8s

# Vagrant 스크립트 다운로드
curl -O https://raw.githubusercontent.com/k8s-1pro/install/main/ground/k8s-1.27/vagrant-2.4.3/Vagrantfile

# Rocky Linux Repo 세팅
curl -O https://raw.githubusercontent.com/k8s-1pro/install/main/ground/k8s-1.27/vagrant-2.4.3/rockylinux-repo.json
vagrant box add rockylinux-repo.json

# Vagrant Disk 설정 Plugin 설치 
vagrant plugin install vagrant-vbguest vagrant-disksize

# Vagrant 실행 (VM생성)
vagrant up
```

   - Vagrant 명령어
     + vagrant up : VM 생성 (최초 VM생성 할때만 사용, 생성 이후 부터 컴퓨터를 껐다 켜거나 했을 때, VM 기동 / 중지는 Virtualbox UI를 사용하는걸 권장)
     + vagrant destroy : VM 삭제 (vagrant up으로 VM 생성 중 에러가 났을 때 이 명령으로 삭제)

   - MobaXterm 설치 (23.1 버전)
     + Download : ```https://download.mobatek.net/2312023031823706/MobaXterm_Portable_v23.1.zip```
     + Site : ```https://mobaxterm.mobatek.net/download-home-edition.html```

   - Master Node로 원격 접속 (Windows)
     + Sessions > New session을 선택해서 접속 세션 생성
     + 최초 id는 root, password는 vagrant
     + 참고 이미지
<div align="center">
<img src="https://github.com/user-attachments/assets/273e0f60-a55f-405a-9b8a-f4daea556a6c">
</div>

   - Pod 확인 
```bash
k get pods -A
```
  - 참고 이미지
<div align="center">
<img src="https://github.com/user-attachments/assets/03f661bc-7e1c-454c-843f-29f429c876b8">
</div>

   - 대시보드 접속
```bash
https://192.168.56.30:30000/#/login
```
<div align="center">
<img src="https://github.com/user-attachments/assets/aebbacc8-ebd8-4c61-a94a-fd9eeb8021f5">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/d928d80a-f382-46db-a914-ccdc0aa4cb48">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/616911eb-04ed-4b83-8271-a3b475779b6d">
</div>

5. 구간별 상태 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/06488b50-1688-4412-8412-b5607b871d36">
</div>

   - config.vm.box = "rockylinux/8" (Rocky Linux 8 버전 사용)
   - config.vm.define "master-node" (VM Name)
     + master.vm.hostname = "k8s-master" (Hostname)
     + master.vm.network = "prviate_network" (Host-Only Network : 내 PC에서만 사용할 수 있는 네트워크 망)
     + ip: "192.168.56.30" (IP를 설정하면, Linux에 해당 IP로 할당)
     + 스크립트를 생성하지 않아도 Vagrant가 기본적으로 생성하는 네트워크 존재 : NAT라는 네트워크 (IP도 알아서 할당해주는데, NAT는 VM과 외부 인터넷을 연결시켜주는 역할) - Containerd Package, Kubernetes Package 다운로드

   - 위 그림의 경우 ```192.168.219```까지 고정이며, 네번쨰 자리부터 1 ~ 255까지 생성 가능한데, 100으로 자동 할당
     + Host-Only Network의 경우에는, ```192.168```까지 고정이며, CIDR를 설정하면 지정한 범위 내 IP 할당 (네트워크 대역폭이 겹칠 때, 공유기와 VirtualBox가 같은 IP를 생성하여 IP 충돌 발생)

   - vb.memory = 4096 (4GB)
   - vb.cpus = 4 (CPU 4코어)
   - 위 설정에 따라 VM의 자원이 할당 

   - 내 PC 네트워크 확인 : 윈도우 > 실행 > cmd 입력 > 확인
```bash
c:\사용자>ipconfig
```
<div align="center">
<img src="https://github.com/user-attachments/assets/0f72f482-9b13-47b7-8a26-41a80b72d573">
</div>

   - PC 자원 확인 : 윈도우 하단 상태바 우클릭 > 작업관리자 > 성능 탭
<div align="center">
<img src="https://github.com/user-attachments/assets/802a4685-de81-4260-9583-c479533cfaf9">
</div>

   - VirtualBox 설치 버전 확인 : Virtualbox 실행 > 도움말 > Virtualbox 정보
<div align="center">
<img src="https://github.com/user-attachments/assets/2bc9505b-072f-4d44-896c-9562b06dcf9c">
</div>

   - Vagrant 설치 버전 확인 : 윈도우 > 실행 > cmd 입력 > 확인
```bash
c:\사용자>vagrant --version
```
<div align="center">
<img src="https://github.com/user-attachments/assets/fe6318f5-35c4-4272-9a49-33603dfb4954">
</div>

   - 원격접속(MobaXterm) 설치 버전 확인 : MobaXterm 실행 > Help > About MobaXterm
<div align="center">
<img src="https://github.com/user-attachments/assets/a5294cd9-78f0-493b-9a9b-ebf574437635">
</div>

   - VirtualBox VM 확인 : Virtualbox 실행 > VM Name 확인 (네이밍 : ```<Vagrant 폴더명>_<VM Name>_<ramdom>```)
<div align="center">
<img src="https://github.com/user-attachments/assets/ae9ee303-f99d-4c90-91e9-96e1d5137a6e"">
</div>

   - VM에 적용된 NAT 확인 : Virtualbox 실행 > k8s_master-node 마우스 우클릭 > 설정 > 네트워크 > 어댑터 1
<div align="center">
<img src="https://github.com/user-attachments/assets/fb0e0a50-c275-4c53-a2fd-4a4e4c1c4a0c">
</div>

   - VM에 적용된 Host-Only Network 확인 : Virtualbox 실행 > k8s_master-node 마우스 우클릭 > 설정 > 네트워크 > 어댑터 2
<div align="center">
<img src="https://github.com/user-attachments/assets/5c3c51c0-b026-4959-aef3-0377f4c337be">
</div>

   - VirtualBox Host-Only cidr 확인 : 파일 > 도구 > Network Manager
<div align="center">
<img src="https://github.com/user-attachments/assets/223efb72-3bd3-4fc8-aa20-e500f90db899">
</div>

   - Rocky Linux 버전 확인 : k8s-master 원격접속 후 명령어 실행
```bash
[root@k8s-master ~]# cat /etc/*-release
```
<div align="center">
<img src="https://github.com/user-attachments/assets/953593e7-b4b4-471a-b50e-362ceb695f7a">
</div>

   - Hostname 확인 : k8s-master 원격접속 후 명령어 실행
```bash
[root@k8s-master ~]# hostname
```
<div align="center">
<img src="https://github.com/user-attachments/assets/5163318a-21a2-4d27-9973-ce9a8541641d">
</div>

   - Network 확인 : k8s-master 원격접속 후 명령어 실행
```bash
[root@k8s-master ~]# ip addr
```
<div align="center">
<img src="https://github.com/user-attachments/assets/3f6e16eb-b140-4bc8-a638-905a6b7a15f4">
</div>

  - 자원(CPU, Memory) 확인 : k8s-master 원격접속 후 명령어 실행
```bash
[root@k8s-master ~]# lscpu
[root@k8s-master ~]# free -h
```
<div align="center">
<img src="https://github.com/user-attachments/assets/c39c6833-d0f3-4ee2-a7d6-c41c6efc1b23">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/d1ab8b12-8f1d-478e-9b74-e610cb237140">
</div>
