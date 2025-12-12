-----
### 실습 - 배포 파이프라인 구축 후 마주하게 되는 고민들
-----
1. item name 입력 및 Pipeline 선택
```
Enter an item name에 [2224-deploy-helm] 입력
Copy form에 [2223-deploy-helm] 입력
[OK] 버튼 클릭
```

2. Configure > Advanced Project Options > Pipeline
```
Definition > SCM > Branches to build > Additional Behaviours > Sparse Checkout paths > Path : 2224
Definition > Script Path : 2224/Jenkinsfile
```

3. 배포 파이프라인 구축 후 마주하게 되는 고민들
<div align="center">
<img src="https://github.com/user-attachments/assets/2376064b-8ba4-4ffb-a10c-5e3e4240c2ae" />
</div>

4. 중요 데이터 암호화 관리
  - 도커 접속 정보 및 쿠버네티스 config를 위한 New credentials 생성
  - Dashboard > Jenkins 관리 > Credentials > System > Global credentials (unrestricted) 에서 [Add Credentials] 클릭 
  - Docker 접속정보 Credentials 등록
<div align="center">
<img src="https://github.com/user-attachments/assets/bd7be376-8381-4ad1-9004-c924a4b6abba" />
</div>

  - Kubernetes config 파일 Credentials 등록
<div align="center">
<img src="https://github.com/user-attachments/assets/66a3fd3b-4123-428e-b3d9-2d69447feee1" />
</div>

  - [파일 선택]을 위해 사전에 CI/CD Server에 있는 config 파일을 내 PC로 복사해 놓아야 함
```bash
# config 파일 위치
[jenkins@cicd-server .kube]$ pwd
/var/lib/jenkins/.kube
[jenkins@cicd-server .kube]$ ls
cache config
```
  - Windows에서 이동 방법 : /var/lib/jenkins/.kube/를 입력 후 config 파일이 나오면 내 PC 바탕화면으로 드래그
<div align="center">
<img alt="image" src="https://github.com/user-attachments/assets/cc6c026f-6ca8-4a12-b227-5f37f7bdf57f" />
</div>

5. Jenkinsfile 내용 
```jenkins
// Docker 사용
steps {
  script{
    withCredentials([usernamePassword(credentialsId: 'docker_password', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
    sh "echo " + '${PASSWORD}' + " | docker login -u " + '${USERNAME}' + " --password-stdin"

// Kubernetes config 사용
steps {
  withCredentials([file(credentialsId: 'k8s_master_config', variable: 'KUBECONFIG')]) {    // 암호화로 관리된 config가 들어감
    sh "kubectl apply -f ./2224/deploy/kubectl/namespace-dev.yaml --kubeconfig " + '${KUBECONFIG}'
    sh "helm upgrade api-tester-2224 ./2224/deploy/helm/api-tester -f ./2224/deploy/helm/api-tester/values-dev.yaml" +
        " -n anotherclass-222-dev --install --kubeconfig " + '${KUBECONFIG}'
```

6. CI/CD Server에서 Docker Logout 및 kubeconfig 삭제
```bash
# Docker Hub Logout 하기
[jenkins@cicd-server ~]$ docker logout
Removing login credentials for https://index.docker.io/v1/
[jenkins@cicd-server .docker]$ cat ~/.docker/config.json
{
        "auths": {}
}

# kubeconfig 파일명 변경하기
[jenkins@cicd-server ~]$ mv ~/.kube/config ~/.kube/config_bak
[jenkins@cicd-server ~]$ kubectl get pods -A
E1219 17:16:27.450820    3778 memcache.go:265] couldn't get current server API group list: <html>...
```

7. 잦은 배포 - Versioning 무의미, 계획된 배포 - Versioning 필수 - Jenkinsfile 내용 
```jenkins
environment {
  APP_VERSION = '1.0.1'
  BUILD_DATE = sh(script: "echo `date +%y%m%d.%d%H%M`", returnStdout: true).trim()
  // 위에 date 포맷 오류가 있어요. %y%m%d.%H%M%S가 맞습니다)
  TAG = "${APP_VERSION}-" + "${BUILD_DATA}"

stage('컨테이너 빌드 및 업로드') {
  steps {
	script{
	  // 도커 빌드
      sh "docker build ./2224/build/docker -t 1pro/api-tester:${TAG}"
      sh "docker push 1pro/api-tester:${TAG}"

stage('헬름 배포') {
  steps {
    withCredentials([file(credentialsId: 'k8s_master_config', variable: 'KUBECONFIG')]) {
      sh "helm upgrade api-tester-2224 ./2224/deploy/helm/api-tester -f ./2224/deploy/helm/api-tester/values-dev.yaml" +
         ...
         " --set image.tag=${TAG}"   // Deployment의 Image에 태그 값 주입
```

8. docker tag 변경 명령 (to latest)
```bash
docker tag 1pro/api-tester:1.0.1-231220.171834 1pro/api-tester:latest
docker push 1pro/api-tester:latest
```

9. 업로드 후 CI/CD Server에 만들어진 이미지 삭제 - Jenkinsfile 내용 
```jenkins
stage('컨테이너 빌드 및 업로드') {
  steps {
	script{
	  // 도커 빌드
      sh "docker build ./${CLASS_NUM}/build/docker -t ${DOCKERHUB_USERNAME}/api-tester:${TAG}"
      sh "docker push ${DOCKERHUB_USERNAME}/api-tester:${TAG}"
      sh "docker rmi ${DOCKERHUB_USERNAME}/api-tester:${TAG}"   // 이미지 삭제
```

10. 네임스페이스는 배포와 별도로 관리 - Jenkinsfile 내용 
```jenkins
stage('네임스페이스 생성') {  // 배포시 apply로 Namespace 생성 or 배포와 별개로 미리 생성 (추후 삭제시 별도 삭제)
  steps {
    withCredentials([file(credentialsId: 'k8s_master_config', variable: 'KUBECONFIG')]) {
      sh "kubectl apply -f ./2224/deploy/kubectl/namespace-dev.yaml --kubeconfig " + '"${KUBECONFIG}"'
...
stage('헬름 배포') {
  steps {
```

11. Helm 부가기능 (배포  후 Pod 업그레이드가 안될 수 있음) - deployment.yaml 내용 
```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    metadata:      
      annotations:  
        rollme: {{ randAlphaNum 5 | quote }} // 항상 새 배포를 위해 랜덤값 적용
```

12. Helm 부가기능 (Pod가 완전 기동 될때까지 결과값 기다림) - helm command 명령어 내용
```jenkins
stage('헬름 배포') {
  steps {
    withCredentials([file(credentialsId: 'k8s_master_config', variable: 'KUBECONFIG')]) {
      sh "helm upgrade api-tester-2224 ./2224/deploy/helm/api-tester -f ./2224/deploy/helm/api-tester/values-dev.yaml" +
         ...
         " --wait --timeout=10m" +  // 최대 10분으로 설정
```

13. 사용 안하는 이미지는 자동 삭제됨 - config.yaml 내용 
```bash
// GC 속성 추가하기
[root@k8s-master ~]# vi /var/lib/kubelet/config.yaml
-----------------------------------
imageMinimumGCAge : 3m // 이미지 생성 후 해당 시간이 지나야 GC 대상이 됨 (Default : 2m)
imageGCHighThresholdPercent : 80 // Disk가 80% 위로 올라가면 GC 수행 (Default : 85)
imageGCLowThresholdPercent : 70 // Disk가 70% 밑으로 떨어지면 GC 수행 안함(Default : 80)
-----------------------------------

// kubelet 재시작
[root@k8s-master ~]# systemctl restart kubelet
```
  - Kubernetes Docs : ```https://kubernetes.io/docs/concepts/architecture/garbage-collection/```

14. [파라미터와 함께 빌드] 실행 후 PROFILE을 [dev]로 선택하고 [빌드하기] 클릭
<div align="center">
<img src="https://github.com/user-attachments/assets/97295c4f-b194-4d54-83e4-4cd83dd85872" />
</div>

15. 실습 후 정리
```bash
# helm 조회
helm list -n anotherclass-222

# helm 삭제
helm uninstall -n anotherclass-222-dev api-tester-2224
helm uninstall -n anotherclass-222-qa api-tester-2224
helm uninstall -n anotherclass-222-prod api-tester-2224

# namespace 삭제
kubectl delete ns anotherclass-222-dev
kubectl delete ns anotherclass-222-qa
kubectl delete ns anotherclass-222-prod
```
