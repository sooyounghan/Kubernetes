-----
### Namespace, ResourceQuota, LimitRange
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/411d4311-a0e0-49c4-9404-1d8e9f707c97">
</div>

1. 사용 상황
   - Kubernetes Cluster : 전체 사용할 수 있는 자원 (일반적으로 메모리, CPU 등이 존재)
     + Cluster 안에는 여러 Namespace들을 만들 수 있고, Namespace 안에는 여러 Pod들을 만들 수 있음
     + 각 Pod는 필요한 자원을 클러스터 자원을 공유해서 사용 : 만약 한 Namespaace 안에 있는 Pod가 클러스터에 남은 자원을 모두  사용해버리면, 다른 Pod 입장에서는 더 이상 쓸 자원이 없어 자원이 필요할 때 문제 발생
     + 이런 문제를 해결하기 위해 ResourceQuota가 존재 : Namespace에 적용하면 네임스페이스마다 최대 한게 설정 가능하며, Pod 자원이 이 한계를 넘을 수 없게 되어, Pod 입장에서는 자원이 부족해지는 문제가 발생하지만, 다른 Namespace의 Pod들에게는 영향을 미치지 않음

   - 위 그림과 같이 한 Pod가 자원 사용량을 너무 크게 해버리면, 다른 Pod들이 이 Namespace에 더 이상 들어올 수 없게 됨
     + 이를 관리하기 위해 LimitRange를 설정해 Namespace에 들어오는 Pod의 크기 제한 가능
     + 이렇게 한 Pod의 자원 사용량이 LimitRange에 설정된 값보다 낮아야 Namespace로 진입 가능
     + 이보다 클 경우에는 Pod가 NameSpace에 진입 불가

   - 두 Object는 Namespace뿐만 아니라 Cluster에도 적용이 가능하여 자원에 대한 전체 제한 가능

<div align="center">
<img src="https://github.com/user-attachments/assets/50725ea6-c848-47b9-a3a3-aefbb4dcaaee">
</div>

2. Namespace
   - 한 Namespace 안에는 같은 타입의 Object들은 이름이 중복될 수 없다는 특징을 가지고 있으므로, 같은 Pod의 이름을 중복해서 생성 불가
     + Object들마다 별도의 UUID가 존재하는데, 한 Namespace 안에는 같은 종류의 Object라면 이름 또한 UUID 같이 유일한 키 역할을 하는 셈
   - 다른 Namespace에 있는 자원과 분리가 되서 관리가 됨
     + Pod와 Service 간 연결은 Pod에는 Label을 달고, Service에는 Selector를 달아서 연결하지만, Namespace는 그러지 않음
     + 물론, Node나 Volume과 같이 모든 Namespace에서 공용으로 사용되는 Object도 존재
   - Namespace을 지우게 되면, 그 안에 있는 자원 모두 지워짐
   - Pod IP로 접근하게 되면, 기본적으로 연결이 되지만, NeworkPolicy라는 Object를 통해 제한 가능
   - yaml 파일
     + Pod나 Service를 생성할 때, 속할 Namespace 지정 (namespace)
     + 두 Object는 Namespace가 서로 다르므로, selector와 label의 값이 일치하더라도 연결이 되지 않ㄹ음

3. ResourceQuota
   - Namespace의 자원 한계를 설정하는 Object
   - Namespace에 제안하고 싶은 자원을 명시해 설정 가능
     + request.memory : 해당 Namespace에 들어갈 Pod들의 전체 Request 자원들을 최대 3GB로 설정
     + limits.memory : 메모리의 한계값은 6GB로 설정
     + 💡 ResourceQuota가 지정된 네임스페이스에 Pod를 만들 때, 해당 Pod는 무조건 지정한 spec을 명시해야 함(request, limits)
       * spec 자체가 없으면 Namespace가 생성되지 않음
     + 또한, 3GB Request 중 2GB를 사용하는 Pod가 있는데, 이 Request를 2GB 더 사용하겠다는 Pod가 들어오면 만들어지지 않음
   - yaml 파일
     + ResourceQuotat를 생성할 때, 할당할 namespace를 설정
     + Pod 속성에 제한할 종류와 자원 한계치가 설정(resoucres.requests, resources.limits)
     + 메모리 뿐만 아니라 CPU, Storage도 존재하며, Object 들의 숫자도 제한할 수 있지만, 모든 Object들을 다 제한할 수 있는 것은 아니지만, Kubernetes 버전이 업데이트되면서 제한할 수 있는 Object의 숫자가 점점 증가되고 있음

  4. LimitRange
     - 각 Pod마다 Namespace에 들어올 수 있는 자원을 확인
       + min : Pod에서 설정되는 메모리의 Limit 값이 1GB가 넘어야 함
       + max : Pod에서 설정되는 메모리의 값이 4GB를 초과할 수 없음
       + maxLimitRequestRatio : 3 - request 값과 limit 값의 비율이 최대 3배를 넘으면 안되는 뜻
         * 예) Pod1의 경우, 설정된 limits의 값은 메모리 5GB이지만, max가 4GB이므로 해당 Pod는 불가
         * 예) Pod2의 경우, request와 limits의 값이 1와 4의 비율인데 3배까지만 허용되므로 불가
     - defaultRequest, defaultLimit : Pod에서 아무런 spec 설정을 하지 않았을 때, 자동으로 이 request와 limit 값이 명시된 값으로 할당
     - yaml 파일
       + LimitRange을 만들 때, 할당할 namespace 명시
       + limit라는 속성의 타입을 컨테이너 별로 제한할 때 옵션 설정 가능
       + Container 타입에는 Pod 단위 설정과 VolumeCliam 단위의 설정이 있는데, 각각의 타입마다 이 지원되는 옵션의 종류가 다르므로 별도 확인이 필요
