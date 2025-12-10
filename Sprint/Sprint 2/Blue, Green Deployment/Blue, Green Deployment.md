-----
### Blue / Green 배포
-----
1. item name 입력 및 Pipeline 선택
```bash
Enter an item name : 2213-jenkins_pipeline-step3
Copy from : 2212-jenkins_pipeline-step2
```
<div align="center">
<img src="https://github.com/user-attachments/assets/f764e59a-d398-4c3b-8cec-b8eeaf7773a8" />
</div>

2. Configure > Additional Behaviours 및 Script Path 수정 후 저장
```
Path : 2213
Script Path : 2213/Jenkinsfile
```
<div align="center">
<img src="https://github.com/user-attachments/assets/2a136f61-cd38-4447-b026-ccf20672f53d" />
</div>

3. Master Node에서 version 조회 시작
```bash
[root@k8s-master ~]# while true; do curl http://192.168.56.30:32213/version; sleep 1; echo '';  done;
```

4. [지금 빌드] 실행 (k8s-1pro로 Repository, Git 모두 설정 / 파라미터도 설정)
5. [수동배포 시작] yes 클릭 - Green이 배포됨
<div align="center">
<img src="https://github.com/user-attachments/assets/c5ab34f9-aec4-49cb-88ed-28105b074dc8" />
</div>

6. [QA 담당자] V2 Service 호출 가능
```bash
curl http://192.168.56.30:32223/version
```

7. [Green 전환] yes 클릭 및 v2로 버전 변경 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/7f7dd88b-f415-4ecd-a524-9334056af788" />
</div>

  - 바로 v2.0.0으로 변경됨
<div align="center">
<img src="https://github.com/user-attachments/assets/152f4a05-5679-4396-88f2-436275d6232c" />
</div>

8. 롤백 여부 선택
<div align="center">
<img src="https://github.com/user-attachments/assets/df65448d-ac2f-4815-93eb-efebfb5d0bff" />
</div>

  - done : Blue 배포 삭제됨
  - rollback : Blue로 다시 전환됨
