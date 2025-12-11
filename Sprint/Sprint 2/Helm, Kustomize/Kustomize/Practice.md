-----
### 다양한 배포 환경을 위한 Kustomize 배포하기
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/f9102e45-3911-498a-afa6-7de478abf695" />
</div>


1. item name 입력 및 Pipeline 선택
```bash
Enter an item name에 [2222-deploy-kustomize] 입력
Copy form에 [2221-deploy-helm] 입력
[OK] 버튼 클릭
```
  
  - Configure > Advanced Project Options > Pipeline
```bash
Definition > SCM > Branches to build > Additional Behaviours > Sparse Checkout paths > Path : 2222
Definition > Script Path : 2222/Jenkinsfile
```

2. [저장] 후 [지금 빌드] 실행 후 [Abort] 그리고 [페이지 새로고침] 하기
<div align="center">
<img src="https://github.com/user-attachments/assets/2f518435-14a2-401e-929c-5ff2b547e82a" />
</div>

   - 최초 실행시엔 매개변수 입력 버튼이 안나옴 [dev, qa, prod]중 dev가 적용됨

3. [파라미터와 함께 빌드] 실행 후 PROFILE을 [dev]로 선택하고 [빌드하기] 클릭
<div align="center">
<img src="https://github.com/user-attachments/assets/2fc0b27b-270b-46ac-94e3-366fd0492bca" />
</div>

4. [Stage View] 결과 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/9c035033-4def-445c-9b89-4f55dda1cd7c" />
</div>

5. 배포 되기전 꼭 배포될 템플릿 Logs 확인하기

6. [파라미터와 함께 빌드] 실행 후 PROFILE을 [qa], [prod]로 선택하고 [빌드하기] 클릭 그리고 [대시보드] 확인

​7. 디렉토리 설명
<div align="center">
<img src="https://github.com/user-attachments/assets/65c420ac-8ba9-4ccb-93f9-260890da3cb7" />
</div>

8. [배포 코드] 확인 : ```https://github.com/k8s-1pro/kubernetes-anotherclass-sprint2/tree/main/2222```
​9. 실습 후 정리
```bash
# namespace 삭제
kubectl delete ns anotherclass-222-dev
kubectl delete ns anotherclass-222-qa
kubectl delete ns anotherclass-222-prod
```
