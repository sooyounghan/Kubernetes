-----
### 이름 떄문에 기대가 너무 컸던 Secret
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/10c78a57-1040-44d0-ac3b-1e41d3e396ad" />
</div>

1. Secret은 중요한 데이터를 관리하기 위한 목적으로 사용
2. Secret은 Configmap과 달리 type이라는 속성 존재 : 기본적으로 넣지 않으면 opaque라는 값으로 설정
   - Opaque(투명)이라는 값이 들어가 Configmap을 써도 되는 모든 상황에 조금 중요한 데이터라는 생각이 되면 타입의 Secret으로 대체 가능 : 즉, Configmap과 동일
   - type: docker-registry
     + data는 docker-username, docker-paasword, docker-email의 값들을 넣어줘야 함
     + 현재 이 컨테이너를 만들 때, 이 이미지는 Public DockerHub에서 가져오는데, 공개가 아닌 Private Docker Registry를 설치하고, 여기서 이미지를 가져오고 싶을 때 사용
     + 즉, Dokcer Registry 타입의 Secret을 만들고, 데이터의 Key-Value로 Private Registry에 대한 정보를 넣어 Pod에 설정
     + Kubernetes가 Pod에 연결되어 있는 Secret 정보를 가지고 이미지를 다운 : 이 때, Pod에 연결되는 속성도 ImagePullSecrets로 다름

   - 결국 사용자 편의에 따라 Custom하게 관리할 수 있는 소스를 제공 : Secret이 Key와 Value를 설정하는 Template 역할이므로, 각 타입에는 정해진 Key가 존재
   - 하지만 이런 type 중에는 데이터 암호화를 제공해주는 타입은 없음

3. 데이터 암호화 방법
   - Secret에 대한 Object 생성을 Pipeline에 전송해 만들지 말고, Kubernetes 클러스터 내에서 직접 만들고 관리 : 접근 제어를 통한 보안 관리 방법
   - 문자 자체적으로 암호화 : 특정 키를 가지고 문자를 암호화해놓고, 이 암호화된 문자를 Secret으로 관리 (실제 비밀번호는 알 수 없음), 단, 복호화해줄 수 있는 로직이 필요 (직접 코딩 또는 WAS 자체 제공)
   - 암호화를 관리해주는 Third-Party 도구 사용 : HashiCorp의 Valut가 대표적
     + 관리자에 한해 개인 아이디나 패스워드로 Value에 접속
     + App이 기동하면서 Third-Party App에 비밀번호를 요청하는데, 관리자가 지정한 Pod에게만 값을 주도록 세팅
   - 두 번째와 세 번째 방법 : 클러스터에 접근하더라도 App의 중요 데이터를 유출되는 것을 막을 수 있음
