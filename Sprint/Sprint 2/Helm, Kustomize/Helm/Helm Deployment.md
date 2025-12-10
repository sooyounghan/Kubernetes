-----
### Helm Deployment (설정 부분 : Sprint 2 76, 77강 참고)
-----
1. Helm(Ver. 3.13.2) 설치
  - Download : ```https://get.helm.sh/helm-v3.13.2-linux-amd64.tar.gz```
  - Site : ```https://helm.sh/docs/intro/install/```
  - Release : ```https://github.com/helm/helm/releases```
  - CI/CD Server에서 helm 설치 (root 유저) - 인텔 PC용
```bash
[root@cicd-server ~]#
curl -O https://get.helm.sh/helm-v3.13.2-linux-amd64.tar.gz
tar -zxvf helm-v3.13.2-linux-amd64.tar.gz
mv linux-amd64/helm /usr/bin/helm
```

  -  확인하기
```bash
# jenkins 유저로 전환해서 확인
[root@cicd-server ~]# su - jenkins -s /bin/bash
[jenkins@cicd-server ~]$ helm
```

  - 템플릿 생성하기 (helm create : 초기 템플릿을 샘플로 생성 / api-tester : 패키지의 차트 이름)
```bash
[jenkins@cicd-server ~]$ helm create api-tester
```

2. Kustomize(Ver. 5.0.1) 설치
  - Download : 별도 설치 필요없음 (kubectl v1.14부터 포함되어 있음) - 현재 v1.27.2 사용중
  - Github : ```https://github.com/kubernetes-sigs/kustomize```
  - Site : ```https://kubectl.docs.kubernetes.io/```
  - kubernetes Docs : ```https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/```
  - Kustomize Version 확인
```bash
kubectl version --client
```

3. New view 만들기
   - [+] 버튼을 눌러서 [새 보기] 만들기 
```
조회명 : 222
Type : List View
```

4. Helm 배포 시작하기 - 2221
   - item name 입력 및 Pipeline 선택
```bash
Enter an item name에 [2221-deploy-helm] 입력
[Pipeline] 선택
[OK] 버튼 클릭
```

  - Configure > General > GitHub project > Project url (```<Github-Username>``` : 본인의 Username으로 변경)
```
Project url : https://github.com/<Github-Username>/kubernetes-anotherclass-sprint2/
```

  - Configure > Advanced Project Options > Pipeline
```bash
Definition : Pipeline script from SCM
Definition > SCM : Git
Definition > SCM > Repositories > Repository URL : https://github.com/<Github-Username>/kubernetes-anotherclass-sprint2.git
Definition > SCM > Branches to build > Branch Specifier : */main
Definition > SCM > Branches to build > Additional Behaviours > Sparse Checkout paths > Path : 2221
Definition > Script Path : 2221/Jenkinsfile
```

   - [저장] 후 [지금 빌드] 실행 그리고 [Stage View] 결과 확인 (배포 되기전 꼭 배포될 템플릿 Logs 확인)
<div align="center">
<img src="https://github.com/user-attachments/assets/ac24b958-b7f3-43ae-9255-9fac527a00d9" />
</div>

   - [배포 코드] 확인 : ```https://github.com/k8s-1pro/kubernetes-anotherclass-sprint2/tree/main/2221```

5. [Helm Package]를 만드는 순서
   - Helm 템플릿 생성 (init)
```bash
# helm 템플릿 생성 
[jenkins@cicd-server ~]$ helm create api-tester
[jenkins@cicd-server ~]$ cd api-tester
[jenkins@cicd-server api-tester]$ ls
charts  Chart.yaml  templates  values.yaml
```

  - 당장 필요없는 내용 삭제하기 (deleting)
<div align="center">
<img src="https://github.com/user-attachments/assets/f4378c9f-5235-4767-b083-8d7f354a6d8d" />
</div>

  - 내 yaml 파일에 맞게 Helm Package 수정 (modify)
    + Service에서 port와 containerPort가 동일하게 들어가는 부분 주의
    + 동적 변수 넣기 (미리 넣기보단 지금 필요한 것만 넣기)
    + 완벽할 필요 없음 : 잘못된 부분이 있으면 배포시 문법에러 발생
<div align="center">
<img src="https://github.com/user-attachments/assets/ebce3dd1-a0f9-4242-8844-d048d4e3a427" />
</div>

   - 내 resource 더 추가 (addition) : configmap, secret 추가 (두 object는 한 App에 여러개 만들 수 있음, name에 하드코딩 필요)
<div align="center">
<img src="https://github.com/user-attachments/assets/aa2fd894-40e2-4291-ac68-654ec2a0b6d7" />
</div>

6. 실습 후 정리
```bash
# helm 조회
helm list -n anotherclass-222
```

```bash
# helm 삭제
helm uninstall -n anotherclass-222 api-tester-2221
```

```bash
# namespace 삭제
kubectl delete ns anotherclass-222
```
