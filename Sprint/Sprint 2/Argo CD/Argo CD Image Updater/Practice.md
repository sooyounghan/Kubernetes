-----
### Argo Image Updater 를 이용한 이미지 자동 배포 (helm)
-----
1. Helm과 Kustomize를 사용했을 때만 동작함
2. Image Updater values-dev.yaml 파일 확인 : ```https://github.com/k8s-1pro/install/blob/main/ground/cicd-server/argo/helm/argocd-image-updater/values-dev.yaml```
```yaml
config:
  argocd:
    serverAddress: "https://argo-cd-argocd-server"

  registries:
    - name: Docker Hub
      api_url: https://registry-1.docker.io
      #credentials: env:DOCKER_HUB_CREDS=username:passowrd
```

3. Image Updater 설치 시 Jenkinsfile에 적용된 Docker연결 내용 : ```https://github.com/k8s-1pro/install/blob/main/ground/cicd-server/argo/Jenkinsfile```
```jenkins
HELM_DEPLOY_COMMAND =  "helm upgrade ${params.TARGET_ARGO} ./${INSTALL_PATH}/helm/${params.TARGET_ARGO} " +
" -f ./${INSTALL_PATH}/helm/${params.TARGET_ARGO}/values-dev.yaml" + " -n argo --install --kubeconfig " + '${KUBECONFIG}' +
" --wait --timeout=10m "  

// image-updater일 경우 도커허브 credentials 주입
if (params.TARGET_ARGO == "argocd-image-updater") {
withCredentials([usernamePassword(credentialsId: 'docker_password', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
    HELM_DEPLOY_COMMAND += " --set config.registries[0].credentials=env:DOCKER_HUB_CREDS="+ '${USERNAME}' + ":" + '${PASSWORD}'
    }
}

sh "eval ${HELM_DEPLOY_COMMAND}"
```
   - Docker Hub 접속 방법 : ```https://argocd-image-updater.readthedocs.io/en/stable/basics/update-methods/```

4. Jenkins에서 배포
   - Dashboard > add-on > deploy-argo > 파라미터와 함께 빌드
```
DEPLOY_TYPE : helm_upgrade
TARGET_ARGO : argocd-image-updater
```

5. Image Updater 동작 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/5d9e4bdf-7f52-47eb-8b9b-c8366e6dbef4" />
</div>

   - Pod 로그 내용 (2분 간격으로 이미지 변경 여부 체크됨)
```
time="2024-01-12T01:13:08Z" level=info msg="Starting image update cycle, considering 1 annotated application(s) for update"
time="2024-01-12T01:13:10Z" level=info msg="Processing results: applications=1 images_considered=1 images_skipped=0 images_updated=0 errors=0"
time="2024-01-12T01:15:11Z" level=info msg="Starting image update cycle, considering 1 annotated application(s) for update"
time="2024-01-12T01:15:13Z" level=info msg="Processing results: applications=1 images_considered=1 images_skipped=0 images_updated=0 errors=0"
time="2024-01-12T01:17:13Z" level=info msg="Starting image update cycle, considering 1 annotated application(s) for update"
time="2024-01-12T01:17:15Z" level=info msg="Processing results: applications=1 images_considered=1 images_skipped=0 images_updated=0 errors=0"
```

6. 자동 배포 설정 방법
   - Application > api-tester-2232 > [DETAILS] 클릭
<div align="center">
<img src="https://github.com/user-attachments/assets/80617330-2ae0-4090-a871-a37f908b3d7d" />
</div>

   - SUMMARY > [EDIT] 클릭​
<div align="center">
<img src="https://github.com/user-attachments/assets/ff759c74-efd5-4562-87bc-4d82414c7b25" />
</div>

   - ANNOTATIONS 내용 입력 후 > [SAVE] 클릭 (```<Dockerhub-Useranme>```은 본인의 Username으로 수정)
```bash
// 도커 허브에서 이미지 대상 지정
argocd-image-updater.argoproj.io/image-list=<alias>=<Dockerhub-Username>/api-tester
// ex) argocd-image-updater.argoproj.io/image-list=1pro-api-tester=1pro/api-tester

// 업데이트 전략 선택
argocd-image-updater.argoproj.io/<alias>.update-strategy=name
// ex) argocd-image-updater.argoproj.io/1pro-api-tester.update-strategy=name

// 태그 정규식 설정 (1.1.1-231220.175735) 
argocd-image-updater.argoproj.io/<alias>.allow-tags=regexp:^1.1.1-[0-9]{6}.[0-9]{6}$
// ex) argocd-image-updater.argoproj.io/1pro-api-tester.allow-tags=regexp:^1.1.1-[0-9]{6}.[0-9]{6}$
```
<div align="center">
<img src="https://github.com/user-attachments/assets/71cab0b2-21a1-4a31-9db7-4b43be286552" />
</div>

   - update-strategy
      + semver : 주어진 이미지 제약 조건에 따라 허용되는 가장 높은 버전으로 업데이트
      + latest : 가장 최근에 생성된 이미지 태그로 업데이트
      + name : 알파벳순으로 정렬된 목록의 마지막 태그로 업데이트
      + digest : 변경 가능한 태그의 최신 푸시 버전으로 업데이트

   - 이미지 태그(tag) 업데이트 방식 설정 : ```https://argocd-image-updater.readthedocs.io/en/stable/basics/update-strategies/```

8. SYNC POLICY에서 [ENABLE AUTO-SYNC] 클릭해서 아래 상태로 변경
<div align="center">
<img  src="https://github.com/user-attachments/assets/536fb7aa-e125-4a44-85ad-71f05f3b6507" />
</div>

   - PRUNE RESOURCES : Git에서 리소스 삭제시 실제 Kubernetes에서도 자원이 삭제됨
   - SELF HEAL : Auto Sync 상태에서 항상 Git에 있는 내용이 적용됨 (이때 ArgoCD나 Kuberentes에서 직접 수정한 내용은 삭제됨)

​7. 도커 빌드 Job 생성
   - 새보기 및 item 생성
```
[새보기] 만들기
조회명 : 223
Type : List View

[item name] 만들기
Enter an item name : 2232-build-docker-for-image-updater
[Pipeline] 선택
[OK] 버튼 클릭
```
   - Configure > General > GitHub project > Project url (```<Github-Useranme>```은 본인의 Username으로 수정)
```
Project url : https://github.com/<Github_Username>/kubernetes-anotherclass-sprint2/
```
​   - Configure > Advanced Project Options > Pipeline > [저장] (```<Github-Useranme>```은 본인의 Username으로 수정)
```
Definition : Pipeline script from SCM
Definition > SCM : Git
Definition > SCM > Repositories > Repository URL : https://github.com/<Github_Username>/kubernetes-anotherclass-sprint2.git
Definition > SCM > Branches to build > Branch Specifier : */main
Definition > SCM > Branches to build > Additional Behaviours > Sparse Checkout paths > Path : 2232
Definition > Script Path : 2232/Jenkinsfile
```

   - Jenkinsfile 파일 확인 : ```https://github.com/k8s-1pro/kubernetes-anotherclass-sprint2/blob/main/2232/Jenkinsfile```
```jenkins
environment {
    ...
    APP_VERSION = '1.1.1'
    BUILD_DATE = sh(script: "echo `date +%y%m%d.%d%H%M`", returnStdout: true).trim()
    // 위에 date 포맷 오류가 있어요. %y%m%d.%H%M%S가 맞습니다)
    TAG = "${APP_VERSION}-" + "${BUILD_DATE}"
}
```
   - [저장] 후 [지금 빌드] 실행 (이 때는 파라미터가 없어서 실행되지 않음)
   - [파라미터와 함께 빌드] 선택 후 본인의 DockerHub와 Github의 Username 입력 후 [빌드] 실행
<div align="center">
<img src="https://github.com/user-attachments/assets/3182972d-cc1e-4cce-8ea0-0763bc331744" />
</div>

   - 작업 후에는 기존 실습 내용을 삭제
<div align="center">
<img src="https://github.com/user-attachments/assets/14a870b8-6c8c-4f85-883b-349372dd18a0" />
<img src="https://github.com/user-attachments/assets/a95d98be-1a21-43f9-8540-2bc38ed98a7b" />
</div>

8. Jenkins 빌드 후 Github로 Push 그리고 ArgoCD로 자동 배포
   - ArgoCD Image Updater를 쓰게되면 배포 Flow가 깔끔해 지지만, 결국 Git의 코드 수정이 필요해지게 됨
   - 비록 ImageUpdater에서 Github에 업데이트(Commit)를 해주는 기능이 있지만, 아직 ArgoCD Image Updater의 쓰임이 안정화 된 건 아니기 때문에 운영에서 사용하기엔 충분한 테스트가 필요함
   - 그렇기 때문에 아직까지는 ImageUpdater보단 기존 방식대로 Jenkins에서 Github에 이미지명을 변경하고 Commit을 하는 스크립트를 만드는 게 더 안정적이라고 볼 수 있음
<div align="center">
<img src="https://github.com/user-attachments/assets/4661e524-6fa8-402c-b7ff-656fd9732e2e" />
</div>

   - 테스트 방법 : ```https://cafe.naver.com/kubeops/553```
