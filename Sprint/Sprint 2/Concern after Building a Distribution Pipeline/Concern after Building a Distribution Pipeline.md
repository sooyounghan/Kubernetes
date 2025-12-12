-----
### 배포 파이프라인 구축 후 마주하게 되는 고민들
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/119c882f-25f6-4b46-8008-6ae0c9dd3279" />
</div>

1. CI / CD 환경에서 Jenkins가 컨테이너 빌드까지 실시하여 DockerHub로 업로드 및 Helm 배포도 실시
   - Helm 배포 시에는 인증서가 필요 및 DokcerHub 업로드를 위해 사전 로그인을 활성화 : 로그인하면 특정 디렉토리에 접속 정보가 들어있는 파일 생성 (Docker가 해당 파일을 읽으므로 로그인을 하지 않아도 이미지를 올릴 수 있음)
     + Docker : config.json 파일은 암호화가 되어있지 않아 DockerHub에 대한 Username, Password 열람 가능
   - kubeconfig도 파일 자체가 관리자 권한 인증서이므로, 관리자 권한으로 Kubernetes Cluster API를 전송할 수 있음
   - 즉, CI / CD 서버에 접속할 수 있다면, DokcerHub와 Kubernetes Cluster가 위험해지므로, 1차적으로 이 서버에 아무나 들어오지 못하도록 접근 보완을 강화해야 함
   - 우선, 디스크에 암호화되지 않는 중요한 정보를 남기지 않아야 하며, 서버마다 보안 점검 툴을 사용해 위배되는 사항을 다 검출
     + 특정 파일에 아이디 / 패스워드가 있거나 중요 인증서들이 방치되어 있지 않은지 처리 : Jenkins의 Credential로 등록 (인증서를 암호화시킨 상태에서 사용할 수 있어 등록한 사람도 등록 이후 이 내용을 볼 수 없으며, Jenkins에서만 Credential을 사용할 수 있는 권한이 있는 사람만 사용 가능)
     + Dokcer 로그인의 경우, Username과 Password가 암호화되지만, 이를 가지고 Docker 로그인을 하고, config.json 파일이 생성 : 따라서, 배포 시마다 DokcerHub로 업로드를 할 때, 로그인을 하면 끝나고 로그아웃을 하는 명령을 넣어줘야 하며, 로그아웃을 하면 파일의 접속 정보는 사라짐
     + Docker Credential Helpers라는 것을 적용하면 파일을 열어도 비밀번호를 알 수 없게 하는 기능 제공

2. 빌드를 계속 진행하게 되면, CI / CD 서버에 컨테이너 이미지들이 쌓임 : 서버에 디스크가 모두 차서 Jenkins가 죽게 되므로, 컨테이너를 업로드한 후 이미지를 잘 삭제해주는 것이 중요
3. Helm으로 App을 배포하는데, App 하나마다 네임스페이스를 만들지 않고, 그룹 개념으로 네임스페이스를 생성할 때 존재
   - 하지만, 네임스페이스는 배포와 별도로 관리해주는 것이 좋음
   - helm install을 할 때도, Namespace가 없으면 생성하는 기능이 존재하지만, 이를 Uninstall할 대, 네임스페이스가 삭제되지 않음 : 의도적으로 네임스페이스가 지워지면 다른 App도 삭제될 수 있으므로 의도적 동작
   - wait라는 옵션 : Helm이 확인 후 Pod가 모두 정상적으로 통신이 되면 배포가 종료됐다는 결과 전송
   - 배포를 했지만 Pod 업그레이드 진행이 안 되는 상황이 존재
     + Kubernetes는 새로 배포되는 Deployment 파일의 template 하위의 내용 변경이 있어야 업그레이드를 시작
     + 하지만, 아무런 변경 사항이 없이 배포를 실시하면, 업그레이드 동작이 발생할 때까지 대기
     + 이를 위해 Helm에서 제공해주는 기능이 있는데 metadata.annotations로 새로 배포 시마다 랜덤 값 생성 : 배포될 때마다 값이 변하므로 Kubernetes는 템플릿에 변경 사항이 있다고 감지를 하고, 항상 업그레이드

4. 이미지 태그
   - 이미지 명 뒤에 버전 태그를 작성 (명명 규칙 상 버전을 의미하는 v를 빼는 것이 맞음)
   - 따라서, 태그 버전을 보면 버전 별로 태그 넘버가 올라가고, latest는 넘버링에 상관없이 가장 마지막 배포 버전
   - 사용하는 입장에서는 최신 버전을 받으려면 latest를 넣거나 태그를 아예 붙이지 않아도 latest 이미지가 다운
   - 개발 환경의 경우 잦은 배포로 인해 버전 관리를 하는 것이 의미가 없지만, 검증이나 운영 환경에서는 새 기능을 모두 만들었거나 패치 버전을 생성할 떄마다 계획된 배포를 해야하므로 Versioning이 필수
     + 개발 환경 : image에 api-test:latest와 같이 latest 태그 작성 및 pullPolicy: Always로 설정 (pullPolicy : Pod에 사용되는 이미지를 어떻게 가져올지에 대한 부분으로 Always로 설정하면 항상 DockerHub에서 이미지를 가져옴)
       * 단점 : DockerHub에 연결이 되지 않으면 Pod가 생성되지 않음
     + 검증 및 운영 환경 : 태그를 명시적으로 작성하며, pullPolicy: IfNotPresent (먼저 노드에 해당 이미지가 있다면 사용하고, 없으면 DockerHub를 확인)
       * 최초의 DockerHub 이미지를 가져와야 되는 건 Always와 동일하지만, IfNotPresent를 사용하는 이유는 운영 중 Pod의 Scale-In / Out이 되기도 하는데, 해당 노드에 이미지가 다운받아져 있는데, 이미지를 재다운로드 하는 것은 비효율적
       * 또한, Alwas를 사용하면 DockerHub에 연결이 되지 않은 상황이 발생하면 무조건 Pod가 안 만들어지지만, IfNotPresent은 적어도 해당 노드에 이미지가 있으면 Pod가 만들어질 확률은 존재
       * 하지만, 개발 환경에서 Always 대신 IfNotPresent를 사용하면 안 됨 : 태그를 latest로 설정한 이유는 항상 DockerHub에서 최신 이미지를 가져오려는 목적인데, IfNotPresent로 설정한다면, 이미 내 노드에 다운받아짐 이전 버전 이미지를 가져오게 됨
     + 즉, 태그를 latest로 설정했다면 pullPolicy는 Always, 태그를 명시적으로 지정할 때는 IfNotPresent로 설정

   - 이렇게 역할을 분리하게 되면, 배포했던 컨테이너 이미지를 Rollback해야되는 상황이 발생 : 개발 환경에서는 latest로 태그를 설정해서 요청에 대응하기 어려움
     + 따라서, 개발 환경에 매 배포마다 새 태그를 설정 : 날짜.시간 + 시퀀스 / pullPolicy는 IfNotPresent로 설정
     + 따라서, Rollback 요청이 오면 원하는 날짜를 찾아 해당 날짜에 맞는 이미지로 Rollback
     + 따라서, 항상 배포할 때마다 이미지 태그가 변하므로 Helm에서 줬던 metadata.annotations 값을 주지 않아도 됨

5. 마지막 배포 버전은 이 버전에 버그가 실행이 안 될 수 있지만 수시로 코드가 반영된 마지막 버전임, 최신 안정화 버전은 마지막으로 운영 환경까지 배포를 안정화했던 버전
   - latest는 최신 안정화 버전의 의미로 가는 것이 좋음

6. 계속 이미지를 다운받게 되면 Kubernetes 노드에 이미지가 누적되어 디스크 공간이 부족할 수 있지만, Kubernetes는 사용하지 않는 이미지를 자동으로 삭제 : Kubernetes Garabage Collector라고 부
