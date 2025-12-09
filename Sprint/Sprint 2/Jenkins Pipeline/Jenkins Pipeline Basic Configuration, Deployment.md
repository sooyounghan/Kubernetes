-----
### Jenkins 파이프라인 기본 구성 및 배포 세분화
-----
1. New view 만들기 (Stage View Plug-in 사전 설치)
   - 이전 작업 [새 보기] 만들어서 정리해 놓기
<div align="center">
<img src="https://github.com/user-attachments/assets/0830d6a7-4906-449b-b0ad-0d6b881dab0a" />
<img src="https://github.com/user-attachments/assets/3c3c91fc-7aaf-4c42-b867-0983db542894" />
<img src="https://github.com/user-attachments/assets/4ac02075-83ec-43bb-8c77-e1513f4ac1cb" />
</div>

  - [새 보기] 만들기 : 이후 작업 추가 없이 [확인]
<div align="center">
<img src="https://github.com/user-attachments/assets/4355cfe7-17a1-4bd6-83ca-8608ef92a71f" />
<img src="https://github.com/user-attachments/assets/736fe948-b83f-46f8-a64f-a1a16d523332" />
</div>

2. Jenkins Pipeline 기본 구성 만들기 (Step 1) - 2211
   - (221) View 선택 후 [새로운 Item] 클릭
<div align="center">
<imgsrc="https://github.com/user-attachments/assets/b19f76f1-f56d-45a6-830a-aff89918cfab" />
</div>

   - item name 입력 및 Pipeline 선택
```
2211-jenkins_pipeline-step1
```
<div align="center">
<img src="https://github.com/user-attachments/assets/5c688776-4b4b-4a2c-9da5-687459e3c3c8" />
</div>

   - Script 복사 : DOCKERHUB_USERNAME과 GITHUB_USERNAME에 본인의 Username 넣
<div align="center">
<img src="https://github.com/user-attachments/assets/744e4cc1-c035-428b-a827-e930f36f1e71" />
</div>

```gradle
pipeline {
    agent any

    tools {
        gradle 'gradle-7.6.1'
        jdk 'jdk-17'
    }

    environment {
        // 본인의 username으로 값.
        DOCKERHUB_USERNAME = ""
        GITHUB_USERNAME = ""

        // 아래 부분 수정(x)
        GITHUB_URL = "https://github.com/${GITHUB_USERNAME}/kubernetes-anotherclass-sprint2.git"
        CLASS_NUM = '2211'
    }

    stages {
        stage('Source Build') {
            steps {
                // 소스파일 체크아웃 (Source Repo는 변경없이 그대로 사용)
                git branch: 'main', url: 'https://github.com/k8s-1pro/kubernetes-anotherclass-api-tester.git'

                // 소스 빌드
                // 755권한 필요 (윈도우에서 Git으로 소스 업로드시 권한은 644)
                sh "chmod +x ./gradlew"
                sh "gradle clean build"
            }
        }

        stage('Container Build') {
            steps {	
                // 릴리즈파일 체크아웃
                checkout scmGit(branches: [[name: '*/main']], 
                    extensions: [[$class: 'SparseCheckoutPaths', 
                    sparseCheckoutPaths: [[path: "/${CLASS_NUM}"]]]], 
                    userRemoteConfigs: [[url: "${GITHUB_URL}"]])

                // jar 파일 복사
                sh "cp ./build/libs/app-0.0.1-SNAPSHOT.jar ./${CLASS_NUM}/build/docker/app-0.0.1-SNAPSHOT.jar"

                // 컨테이너 빌드 및 업로드
                sh "docker build -t ${DOCKERHUB_USERNAME}/api-tester:v1.0.0 ./${CLASS_NUM}/build/docker"
                // 영상과 달리 if문이 없어지고 항상 본인의 Docker Hub에서 빌드가 되도록 수정 됨
                sh "docker push ${DOCKERHUB_USERNAME}/api-tester:v1.0.0"
            }
        }

        stage('K8S Deploy') {
            steps {
                // 쿠버네티스 배포 
                sh "kubectl apply -f ./${CLASS_NUM}/deploy/k8s/namespace.yaml"
				sh "kubectl apply -f ./${CLASS_NUM}/deploy/k8s/configmap.yaml"
				sh "kubectl apply -f ./${CLASS_NUM}/deploy/k8s/secret.yaml"
				sh "kubectl apply -f ./${CLASS_NUM}/deploy/k8s/service.yaml"
				sh "kubectl apply -f ./${CLASS_NUM}/deploy/k8s/deployment.yaml"
            }
        }
    }
}
```

  - Github에서 deployment.yaml 파일의 Image 수정 : image에서 1pro/api-tester:v1.0.0를 ```<Dockerhub-Username>/api-tester:v1.0.0```로 수정
<div align="center">
<img src="https://github.com/user-attachments/assets/9f3b7305-a049-41ba-a11e-2715f5ebbc5c" />
</div>

   - [지금 빌드] 샐행 후 Stage View 결과 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/e18fa7cf-a51d-433d-bcb6-8e4bfa994ead" />
</div>

   - Github 에러시 [Sync fork] 할게 있는지 꼭 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/c5556ae5-fa5e-4696-b8e1-a7b835a53cab" />
</div>

   - Jenkins Pipeline 구조 이해
<div align="center">
<img src="https://github.com/user-attachments/assets/c4b1d8eb-7012-494a-b5ac-17ebcc71d55d" />
</div>

  - Agent (파이프라인 안에서 컨테이너 실행 / Docker 빌드 / YAML 파일도 배포 가능하며, Agent 형태로 제공)
    + agent any : 사용 가능한 에이전트에서 파이프라인 Stage를 실행, Master나 Salve 아무곳에서나 Stage가 실행됨
    + agent label(node) : 지정된 레이블(노드)에서 Stage가 실행
    + agent docker : Docker 빌드를 제공해주는 Agent 사용
    + agent dockerfile : Dockerfile 을 직접 쓸 수 있는 Agent 사용

  - node('slave') ... 로 시작하는 부분 : Jenkins를 Master-Slave 구조로 설치하게 됐을 때, Slave에서 스크립트를 돌리겠다는 의미로, 한 번에 여러 스크립트를 가동해야 할 때, Load를 분산시키려고 하는 것

  - Jenkins Docs : ```https://www.jenkins.io/doc/book/pipeline/syntax/```

3. Github 연결 및 파이프라인 세분화 (Step 2) - 2212
   - item name 입력 및 Pipeline 선택
```
2212-jenkins_pipeline-step2
```
<div align="center">
<img src="https://github.com/user-attachments/assets/54699151-46d9-4a9f-ab58-f08e48236a8d" />
</div>

   - Configure > General > GitHub project에 Project url 입력 (```<Github-Username>``` : 본인의 Username으로 변경)
```
Project url : https://github.com/<Github-Username>/kubernetes-anotherclass-sprint2/
```
<div align="center">
<img src="https://github.com/user-attachments/assets/9c2a2fac-b878-4592-925d-82bc98a309ec" />
</div>

   - Configure > Advanced Project Options > Pipeline 구성 (```<Github-Username>``` : 본인의 Username으로 변경)
```
Repository URL : https://github.com/<Github-Username>/kubernetes-anotherclass-sprint2.git
```
<div align="center">
<img src="https://github.com/user-attachments/assets/5e75edba-67e5-42ff-8c4d-acccf862e7d8" />
</div>

```bash
Branch Specifier : */main
Path : 2212
Script Path : 2212/Jenkinsfile
```
<div align="center">
<img src="https://github.com/user-attachments/assets/83fcc889-65c8-45c6-a579-66e4435a92b2" />
</div>

   - [저장] 후 [지금 빌드] 실행 (이 때는 파라미터가 없어서 실행되지 않음) : 해당 내용에 대해서는 Jenkinsfile의 parameters와 environment 부분의 주석을 확인
   - [파라미터와 함께 빌드] 선택 후 본인의 DockerHub와 Github의 Username 입력 후 [빌드] 실행
<div align="center">
<img src="https://github.com/user-attachments/assets/720f08bd-6ff6-41eb-ad28-772442a12ff9" />
</div>

   - Stage View 결과 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/a435ed14-4657-45bb-8c85-46157036c594" />
</div>

   - 이 때, deployment.yaml 파일에서 image에서 1pro/api-tester:v1.0.0를 ```<Dockerhub-Username>/api-tester:v1.0.0```로 수정해야 하지만 Pipeline을 확인하는 게 핵심이라 넘어가면 됨
     + Blue/Green 배포시 v2.0.0 버전의 이미지가 필요하기 때문에 변경하면, 이미지가 없다는 에러가 발생하게 됨

   - 젠킨스 스크립트 확인 : ```https://github.com/k8s-1pro/kubernetes-anotherclass-sprint2/blob/main/2212/Jenkinsfile```
   - 실습 후 정리 : 실습 후 다음 실습에 방해가 되지 않도록 [kubectl delete ns anotherclass-221] 입력
