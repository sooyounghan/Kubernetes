-----
### 다양한 배포 환경을 위한 Helm 배포하기
-----
1. item name 입력 및 Pipeline 선택
```
Enter an item name에 [2223-deploy-helm] 입력
Copy form에 [2222-deploy-kustomize] 입력
[OK] 버튼 클릭
```

2. Configure > Advanced Project Options > Pipeline
```
Definition > SCM > Branches to build > Additional Behaviours > Sparse Checkout paths > Path : 2223
Definition > Script Path : 2223/Jenkinsfile
```

3. [저장] 후 [지금 빌드] 실행 후 [Abort] 그리고 [페이지 새로고침] 하기
<div align="center">
<img src="https://github.com/user-attachments/assets/1b19d580-34be-4d52-bafb-bc629bd24774" />
</div>

  - 최초 실행시엔 매개변수 입력 버튼이 안나옴 [dev, qa, prod]중 dev가 적용됨

4. [파라미터와 함께 빌드] 실행 후 PROFILE을 [dev]로 선택하고 [빌드하기] 클릭
<div align="center">
<img src="https://github.com/user-attachments/assets/52ed2539-aa19-48c6-b795-0bea1599106d" />
</div>

5. [Stage View] 결과 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/a0601ddc-45f6-4676-ae96-f621ae003a04" />
</div>

  - 배포 되기전 꼭 배포될 템플릿 Logs 확인하기

6. [파라미터와 함께 빌드] 실행 후 PROFILE을 [qa], [prod]로 선택하고 [빌드하기] 클릭 그리고 [대시보드] 확인

​7. 환경별 배포 설명
8. [배포 코드] 확인 : ```https://github.com/k8s-1pro/kubernetes-anotherclass-sprint2/tree/main/2223```
9. 실습 후 정리
```bash
# helm 조회
helm list -n anotherclass-222

# helm 삭제
helm uninstall -n anotherclass-222-dev api-tester-2223
helm uninstall -n anotherclass-222-qa api-tester-2223
helm uninstall -n anotherclass-222-prod api-tester-2223

# namespace 삭제
kubectl delete ns anotherclass-222-dev
kubectl delete ns anotherclass-222-qa
kubectl delete ns anotherclass-222-prod
```
