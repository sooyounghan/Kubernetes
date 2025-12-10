-----
### 자동 배포 만들어보기
-----
1. Blue/Green 자동 배포 Script 만들기 (Step 4) - 2214
   - item name 입력 및 Pipeline 선택
```
Enter an item name : 2214-jenkins_pipeline-step4
Copy from : 2213-jenkins_pipeline-step3
```
<div align="center">
<img src="https://github.com/user-attachments/assets/c7da506d-a0e0-46ad-aa98-f18838c35b95" />
</div>

  - Configure > Additional Behaviours 및 Script Path 수정 후 저장
```
Path : 2214
Script Path : 2214/Jenkinsfile
```
<div align="center">
<img src="https://github.com/user-attachments/assets/30727f97-546b-4122-b38a-65a778328af6" />
</div>

   - Master Node에서 version 조회 시작
```bash
while true; do curl http://192.168.56.30:32214/version; sleep 1; echo '';  done;
```

   - [지금 빌드] 실행
   - [자동배포 시작] yes 클릭
<div align="center">
<img src="https://github.com/user-attachments/assets/ad383efd-b6cc-4506-a69f-89a99ffc739c" />
</div>

   - 진행 과정 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/dcd879bd-c6c7-4653-819e-913928b0c485" />
</div>

   - [Green 배포 확인중] v2.0.0으로 변경됨 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/906f5463-7200-46f9-beff-92c81cb2a188" />
</div>

   - Jenkins 스크립트 확인 : ```https://github.com/k8s-1pro/kubernetes-anotherclass-sprint2/blob/main/2214/Jenkinsfile```
   - 실습 후 다음 실습에 방해가 되지 않도록 [kubectl delete ns anotherclass-221] 실행

2. 리소스 변경 사항 리뷰
<div align="center">
<img src="https://github.com/user-attachments/assets/d5cdbef3-b9a6-40b0-a8be-64fefaf5c5b1" />
</div>

   - Green Deployment 생성 (네이밍, version, blue-green-no 고려)
   - Green Pod에서 Ready 상태 확인
   - Service의 Selector (blue-green-no)를 2로 변경하여 트래픽 Green으로 전환
   - Blue Deployment 삭제 및 관련 모든 리소스의 레이블 정보 변경 (version)

3. Jenkins 파일 확인
```jenkins
pipeline {
    agent any

    parameters {
        // GitHub  사용자명 입력
        string(name: 'GITHUB_USERNAME',  defaultValue: '', description: 'GitHub  사용자명을 입력하세요.')
    }

    environment {
        // 전역값을 넣어 두시면 위 parameters 입력이 필요 없음 (전체 Jenkinfile에서 해당 내용을 모두 수정해 놓으면 좋음)
        // GITHUB_USERNAME = ""

        // 아래 부분 수정(x)
        GITHUB_URL = "https://github.com/${GITHUB_USERNAME}/kubernetes-anotherclass-sprint2.git"
        CLASS_NUM = '2214'
    }


    stages {

        stage('Username 확인') {
            steps {
                script {
                    if (!env.GITHUB_USERNAME?.trim()) {
                        error "[파라미터와 함께 빌드]에 GITHUB_USERNAME를 본인의 username으로 입력해 주세요! 매번 입력이 번거롭다면 Jenkinsfile에서 parameters에 입력 항목을 삭제 하시고 environment에 전역값을 넣은 후 해당 조건문은 삭제해 주세요. "
                    }
                }
            }
        }

        stage('릴리즈파일 체크아웃') {
            steps {
                checkout scmGit(branches: [[name: '*/main']],
                        extensions: [[$class: 'SparseCheckoutPaths',
                                      sparseCheckoutPaths: [[path: "/${CLASS_NUM}"]]]],
                        userRemoteConfigs: [[url: "${GITHUB_URL}"]])
            }
        }

        stage('쿠버네티스 Blue배포') {
            steps {
                sh "kubectl apply -f ./${CLASS_NUM}/deploy/k8s/blue/namespace.yaml"
                sh "kubectl apply -f ./${CLASS_NUM}/deploy/k8s/blue/configmap.yaml"
                sh "kubectl apply -f ./${CLASS_NUM}/deploy/k8s/blue/secret.yaml"
                sh "kubectl apply -f ./${CLASS_NUM}/deploy/k8s/blue/service.yaml"
                sh "kubectl apply -f ./${CLASS_NUM}/deploy/k8s/blue/deployment.yaml"
            }
        }

        stage('자동배포 시작') {
            steps {
                input message: '자동배포 시작', ok: "Yes"
            }
        }

        stage('쿠버네티스 Green배포') {
            steps {
                sh "kubectl apply -f ./${CLASS_NUM}/deploy/k8s/green/deployment.yaml"
            }
        }

        stage('Green 배포 확인중') {
            steps {
                script {
                    def returnValue
                    while (returnValue != "true true"){
                        returnValue = sh(returnStdout: true, encoding: 'UTF-8', script: "kubectl get -n anotherclass-221 pods -l instance='api-tester-2214',blue-green-no='2' -o jsonpath='{.items[*].status.containerStatuses[*].ready}'")
                        echo "${returnValue}"
                        sleep 5
                    }
                }
            }
        }

        stage('Green 전환 완료') {
            steps {
                sh "kubectl patch -n anotherclass-221 svc api-tester-2214 -p '{\"spec\": {\"selector\": {\"blue-green-no\": \"2\"}}}'"
            }
        }

        stage('Blue 삭제') {
            steps {
                sh "kubectl delete -f ./${CLASS_NUM}/deploy/k8s/blue/deployment.yaml"
                sh "kubectl patch -n anotherclass-221 svc api-tester-2214 -p '{\"metadata\": {\"labels\": {\"version\": \"2.0.0\"}}}'"
                sh "kubectl patch -n anotherclass-221 cm api-tester-2214-properties -p '{\"metadata\": {\"labels\": {\"version\": \"2.0.0\"}}}'"
                sh "kubectl patch -n anotherclass-221 secret api-tester-2214-postgresql -p '{\"metadata\": {\"labels\": {\"version\": \"2.0.0\"}}}'"
            }
        }
    }
}
```
