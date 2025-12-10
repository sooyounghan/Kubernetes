-----
### Blue / Green 배포 시 고려해야 할 요소
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/54225ddf-8b01-4b6e-a837-10f223eb8967" />
</div>

1. Deployment 이름 뒤 추가적으로 Sequence가 붙음 : Blue / Green 배포를 고려한 Deployment 네이밍 필요
2. Blue / Green 배포를 위한 추가 labels 및 selector 필요
3. Blue / Green 수동 배포 해보기
<div align="center">
<img src="https://github.com/user-attachments/assets/6efdf0fc-8ace-404d-9d77-99513dd8e559" />
</div>

   - name과 instance 명을 같게 설정
   - Green Deployment 생성
     + 테스트를 위한 Service도 같이 생성
     + 새 Service는 Green Pod에 연결이 되도록 설정, Configmap과 Secret는 Blue Service의 Selector(blue-green-no)를 2로 변경해 트래픽을 Green으로 전환
     + 변경이 되면 트래픽이 전환되는 것이며, Green Deployment에서 문제가 없다고 판단되면 마지막으로 Blue Deployment와 Green Service를 삭제 및 모든 리소스의 레이블 정보 변경(version)
     + 공통 리소스에도 버전 정보처럼 Green Deployment의 정보를 따라서 변경해줘야 되는 레이블들도 꼼꼼하게 잘 변경
     + Green 배포에 문제가 생기면 Rollback (Blue 리소스 삭제 전에 blue-green-no를 1로 변경) 
