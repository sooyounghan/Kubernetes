-----
### Authorization -  RBAC, Role, Role Binding
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/65d0385e-38f1-4eef-bd6b-10d0cf2a118c">
</div>

1. 역할 기반으로 권한을 부여하는 방법
2. Kubernetes에는 Node나 PV, Namespace 등과 같이 Cluster 단위로 관리되는 자원과 Pod와 Service 같이 Namespace 단위로 관리 되는 자원으로 나눌 수 있음
   - Namespace를 만들면, 자동적으로 Service Account가 만들어지기도 하고, 추가적으로 더 생성 가능
   - Service Account Role과 Role Binding을 어떻게 설정하느냐에 따라 해당 Service Account는 네임 스페이스 내 자원만 접근할 수도 있고, Cluster 자원에도 접근할 수 있음

3. Role : 여러 개 만들 수 있고, 각 Role에는 Namespace 내에 있는 자원에 대해서 조회만 가능하거나 생성만 가능하도록 권한을 줄 수 있음
4. Role Binding : Role과 Service Account를 연결해주는 역할로, Role은 하나만 지정할 수 있고, Service Account는 여러 개 지정 가능
   - Service Account(SA)와 SA1은 Role 1에 지정되어 있는 권한으로 API 서버에 접근 가능
   - 따라서, Namespace 내에 권한을 보유할 때 Role과 Role Binding을 여러 개 만들어서 관리하면 됨

5. Service Account에서 Cluster 자원을 접근하고 싶을 때는, Cluster Role과 Cluster Binding이 존재
   - Cluster Role : Cluster 단위 Object들을 지정할 수 있으며, 기능은 동일
   - Cluster Role Binding : Service Account를 추가하면, Namespace A에 있는 Service Account에서도 Cluster 자원에 접근할 수 있는 권한을 얻게 됨
   - Namespace 내 Role Binding이 Service Account와 연결이 되고, Role을 지정할 때, Namespace 내 Role이 아닌 Cluster Role을 지정할 수 있는데, Service Account는 Cluster 자원에는 접근하지 못하고, 자신의 Namespace 내에 있는 자원만 사용이 가능
   - Cluster Role을 사용하는 이유는 모든 Namespace마다 똑같은 Role을 부여하고 관리해야 되는 상황에서 각각의 Namespace 마다 똑같은 내용의 Role을 만들게 되면, Role에 대한 변경이 필요할 때 모든 Role을 수정해야 됨
     + 따라서, Cluster Role 하나만 생성하여 모든 Namespace에 있는 Role Binding이 Cluster Role을 지저할 수 있으면, 권한에 대해 변경사항이 있을 때, 하나만 수정하면 되서 관리가 수월해짐
     + 따라서, 이런 방식은 모든 Namespace에 같은 권한을 만들어 관리할 때 유용

6. Namespace을 만들면, 서비스 계정과 Token Key가 담겨져 있는 Service가 존재
   - apiGroups와 resources 속성이 Pod일 경우 : Pod는 Core API이므로, apiGroups에 넣지 않아도 되며, resoucres가 job일 경우, 해당 API 그룹에 넣어줘야 함
   - verbs는 속성으로 조회만 가능하도록 설정
   - Role Binding을 만들고, roleRef라는 속성에 Role을 연결하고, subjects라는 속성에 Service Account를 연결
   - 이렇게 연결이 되면, Secret의 token 값을 가지고, API 서버에 접근할 수 있고, Token의 권한에 따라 Namespace 내 Pod만 조회

7. 새로운 네임스페이스는 Service Account를 Admin 권한처럼 모든 클러스터 자원에 접근할 수 있도록 설정
   - ClusterRole에 모든 것이 가능하도록 설정
   - ClusterRoleBinding을 만들어서 Service Account 연결
   - 이 토큰으로 API 서버에 접근을 하면 다른 Namespace에 있는 자원과 Cluster 단위 자원에서도 조회 및 생성 가능
