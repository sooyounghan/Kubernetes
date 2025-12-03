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
