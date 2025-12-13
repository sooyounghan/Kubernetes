-----
### ArgoCD 설치 및 배포 (kubectl, helm) - 사전 작업
-----
1. Argo Site : ```https://argoproj.github.io/```
2. Argo Github : ```https://github.com/argoproj```
​3. ArgoCD(Ver.2.9.3) 설치 (k8s V1.25 ~ 28 지원) 
   - Docs : ```https://argo-cd.readthedocs.io/en/stable/```
   - Artifacthub : ```https://artifacthub.io/packages/helm/argo/argo-cd/5.52.1```
4. ArgoCD Image Updater(V.0.9.2) 설치
   - Docs : ```https://argocd-image-updater.readthedocs.io/en/stable/```
   - Artifacthub : ```https://artifacthub.io/packages/helm/argo/argocd-image-updater/0.9.2```
5. Argo Rollouts(Ver. 1.6.4) 설치
   - Docs : ```https://argoproj.github.io/argo-rollouts/```
   - Artifacthub : ```https://artifacthub.io/packages/helm/argo/argo-rollouts/2.34.1```
6. Workflows : ```https://argoproj.github.io/argo-workflows/```
7. Events : ```https://argoproj.github.io/argo-events/```

8. 사전에 작업해 놓은 내용
```bash
# helm이 설치돼 있는 서버에서 작업
# helm 레포지토리(argo-cd) 설정 및 다운로드
helm repo add argo https://argoproj.github.io/argo-helm
helm pull argo/argo-cd --version 5.52.1
helm pull argo/argocd-image-updater --version 0.9.2
helm pull argo/argo-rollouts --version 2.34.1

# 압축 해제
tar -xf argo-cd-5.52.1.tgz
tar -xf argocd-image-updater-0.9.2.tgz
tar -xf argo-rollouts-2.34.1.tgz

# 내용 확인
ls argo*
------
argo-cd-5.52.1.tgz  argocd-image-updater-0.9.2.tgz  argo-rollouts-2.34.1.tgz
argo-cd:
Chart.lock  charts  Chart.yaml  README.md  templates  values.yaml
argocd-image-updater:
Chart.yaml  README.md  templates  values.yaml
argo-rollouts:
Chart.yaml  README.md  templates  values.yaml

# helm package를 Github로 업로드
https://github.com/k8s-1pro/install/tree/main/ground/cicd-server/argo/helm/argo-cd
https://github.com/k8s-1pro/install/tree/main/ground/cicd-server/argo/helm/argo-image-updater
https://github.com/k8s-1pro/install/tree/main/ground/cicd-server/argo/helm/argo-rollouts
```

-----
### ArgoCD 설치 및 배포 (kubectl)
-----
1. ArgoCD 설치하기
   - [+] 버튼을 눌러서 [새 보기] 만들기 
```bash
조회명 : add-on
Type : List View
```
   - item name 입력 및 Pipeline 선택
```bash
Enter an item name에 [deploy-argo] 입력
[Pipeline] 선택
[OK] 버튼 클릭
```
   - Configure > General > GitHub project > Project url (설치 Repo는 별도로 있기 때문에, 여기선 Username 변경없음)
```bash
Project url : https://github.com/k8s-1pro/install/
```
   - Configure > Pipeline
```bash
Definition : Pipeline script from SCM
Definition > SCM : Git
Definition > SCM > Repositories > Repository URL : https://github.com/k8s-1pro/install.git
Definition > SCM > Branches to build > Branch Specifier : */main
Definition > SCM > Branches to build > Additional Behaviours > Sparse Checkout paths > Path : ground/cicd-server/argo
Definition > Script Path : ground/cicd-server/argo/Jenkinsfile
```
  - Helm과 Kustomize 비교하며 사용하기에서 [3-3-1-1. 도커 접속 정보 및 쿠버네티스 config를 위한 New credentials 생성]이 사전에 설정되어 있어야 아래 빌드가 정상 동작

2. [저장] 후 [파라미터와 함께 빌드] 실행
   - Namespace 생성
<div align="center">
<img  src="https://github.com/user-attachments/assets/3242eb3e-aff3-44fc-88fb-9b6036e7034a" />
</div>

   - Argo CD 배포
<div align="center">
<img src="https://github.com/user-attachments/assets/fff44594-fda1-418f-a4eb-8c336fc9dea8" />
</div>

   - Argocd Image Updater 배포
   - Argo Rollouts 배포

3. ArgoCD 접속
```bash
https://192.168.56.30:30002/login
```

<div align="center">
<img src="https://github.com/user-attachments/assets/1d9cf9dc-9fa7-43d4-857d-e4b0b2cca735" />
</div>

   - ArgoCD 관리자 비밀번호 확인 (CI/CD or Mastser server)
```bash
[jenkins@cicd-server ~]$ kubectl get -n argo secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d
L5DwRCgwiHD7CX
```

   - ArgoCD 로그인 (Username : admin / Password : L5DwRCgwiHD7CXDb) / 로그인 이후 [User Info] > [UPDATE PASSOWRD] 에서 변경 가능

4. App 배포하기 (kubectl) - 2231
<div align="center">
<img src="https://github.com/user-attachments/assets/654ba7e4-b787-49f0-92fd-fff0da060c0b" />
</div>

   - App 생성 하기 - [+ NEW APP] 클릭
   - GENERAL
```bash
Application Name : api-tester-2231
Project Name : default
SYNC POLICY : Manual
[체크] AUTO-CREATE-NAMESPACE
```
<div align="center">
<img src="https://github.com/user-attachments/assets/030c4423-9e2c-4b69-a6a6-fa720c0b08ab" />
</div>

   - SYNC OPTIONS
     + SKIP SCHEMA VALIDATION : 매니패스트에 대한 yaml 스키마 유효성 검사를 건너뛰고 배포 (kubectl apply --validate=false)
     + PRUNE LAST : 동기화 작업이 끝난 이후에 Prune(git에 없는 리소스를 제거하는 작업)를 동작시킴
     + RESPECT IGNORE DIFFERENCES : 동기화 상태에서 특정 상태의 필드를 무시하도록 함
     + AUTO-CREATE NAMESPACE : 클러스터에 네임스페이스가 없을 시 argocd에 입력한 이름으로 자동 생성
     + APPLY OUT OF SYNC ONLY : 현재 동기화 상태가 아닌 리소스만 배포
     + SERVER-SIDE APPLY  : 쿠버네티스 서버에서 제공하는 Server-side Apply API 기능 활성화 (레퍼런스 참조)

  - PRUNE PROPAGATION POLICY ​(레퍼런스 참조)
     + foreground : 부모(소유자, ex. deployment) 자원을 먼저 삭제함
     + background : 자식(종속자, ex. pod) 자원을 먼저 삭제함
     + orphan : 고아(소유자는 삭제됐지만, 종속자가 삭제되지 않은 경우) 자원을 삭제함

  - 참고자료
    + ```https://kubernetes.io/docs/reference/using-api/server-side-apply/```
    + ```https://kubernetes.io/ko/docs/concepts/architecture/garbage-collection/```
    + ```https://kubernetes.io/ko/docs/tasks/administer-cluster/use-cascading-deletion/#set-orphan-deletion-policy```

  - SOURCE : 본인의 GithubHub Username 입력
```bash
Repository URL : https://github.com/<Github-Username>/kubernetes-anotherclass-sprint2.git
Revision : main
Path : 2231/deploy/k8s​
```
<div align="center">
<img src="https://github.com/user-attachments/assets/2dae6aa4-d26a-4edf-8dad-6592905b1acd" />
</div>

  - DESTINATION
```bash
Cluster URL : https://kubernetes.default.svc
Namespace : anotherclass-223
[선택] Directory
```
<div align="center">
<img src="https://github.com/user-attachments/assets/4309803b-ecea-47a6-a391-8341730d80bd" />
</div>

  - 화면 상단 [CREATE] 클릭

5. 배포하기 - [SYNC] 클릭 > [SYNCHRONIZE] 클릭
<div align="center">
<img src="https://github.com/user-attachments/assets/0bdfc179-a085-49f5-9671-e07ca5e24dc5" />
<img src="https://github.com/user-attachments/assets/492fda71-5cc4-4b8a-a117-863549f39bdf" />
</div>

   - PRUNE : GIt에서 자원 삭제 후 배포시 K8S에서는 삭제되지 않으나, 해당 옵션을 선택하면 삭제시킴
   - FORCE : --force 옵션으로 리소스 삭제
   - APPLY ONLY : ArgoCD의 Pre/Post Hook은 사용 안함 (리소스만 배포)
   - DRY RUN : 테스트 배포 (배포에 에러가 있는지 한번 확인해 볼때 사용)

6. 배포 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/da216129-382b-4c88-a3c4-c08389735ba4" />
</div>

-----
### ArgoCD 설치 및 배포 (helm)
-----
​1. App 생성 하기 - [+ NEW APP] 클릭
   - GENERAL
```bash
Application Name : api-tester-2232
Project Name : default
SYNC POLICY : Manual
```

   - SOURCE : 본인의 GithubHub Username 입력
```bash
Repository URL : https://github.com/<Github-Username>/kubernetes-anotherclass-sprint2.git
Revision : main
Path : 2232/deploy/helm/api-tester
```

   - DESTINATION
```bash
Cluster URL : https://kubernetes.default.svc
Namespace : anotherclass-223
```

 2. HELM 선택 후 Values files 지정
```bash
VALUES FILES : values-dev.yaml [입력 후 엔터]
```
<div align="center">
<img src="https://github.com/user-attachments/assets/3fea992d-4206-4842-8e65-8df18322093e" />
</div>

3. 화면 상단 [CREATE] 클릭
   - 배포하기 - [SYNC] 클릭 > [SYNCHRONIZE] 클릭
   - 배포 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/a599e490-6040-439a-8b1f-e538112da25d" />
</div>

