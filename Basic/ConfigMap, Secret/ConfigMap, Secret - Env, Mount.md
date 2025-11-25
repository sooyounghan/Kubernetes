-----
### ConfigMap, Secret - Env, Mount
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/34ff4c1e-c16c-4c97-af6d-8ddfd7ea9b7c">
</div>

1. 사용해야 되는 상황
   - 개발(Dev) 환경과 운영(Production) 환경이 존재
   - A라는 서비스가 존재하는데, 이 서비스에는 일반 접근과 보안 접근을 지원
     + 개발 환경에서는 이 보안 접근을 해제할 수 있는 옵션(SSH)이 존재하며, 보안 접근을 한다면 접근 유저와 Key 세팅이 가능
     + 상용 환경으로 배포해야 한다면, 값이 변경해야하며 보안 접속 설정을 변경
     + 하지만 이 값은 컨테이너 안에 있는 서비스 이미지에 들어있는 값이므로 개발 환경과 상용 환경의 컨테이너 이미질르 각각 관리하겠다는 의미
     + 따라서, 환경에 따라 변하는 값들은 외부에서 결정할 수 있도록 도와주는 것이 ConfigMap과 Secret Object
       * 분리해야 되는 일반적인 상수들을 모아 ConfigMap을 만들고, Key와 같이 보안적 관리가 필요한 값을 Secert에 저장
       * 리소스 생성 시, 두 Object를 연결할 수 있는데 연결하게 되면, 해당 컨테이너의 환경 변수에 값이 들어감
     + A 서비스 입장에서는 이 환경 변수의 값을 읽어서 로직을 처리하게 되면 위와 동일하며, 이런 형식으로 이미지 하나만 만들어 놓으면 상용과 개발 환경에 맞게 사용 가능
   - 상용 환경에서는 ConfigMap과 Secret의 데이터만 변경해주면 동일 컨테이너 이미지를 사용해 원하는 기능 추가 가능

<div align="center">
<img src="https://github.com/user-attachments/assets/7622c6fa-ad3d-457d-899b-fa886db74532">
</div>

2. 사용 방법
   - ConfigMap과 Secret을 만들 때, 데이터로 상수를 넣을 수 있고, 파일을 넣을 수 있음
   - 파일을 넣을 때는 환경 변수로 세팅하는 것이 아닌 Volume을 Mount해서 사용 할 수 있음
   - 가장 기본적인 형태 : 상수를 넣는 방법
     + ConfigMap은 Key와 Value로 구성
     + 따라서, 필요한 상수들을 정의해 놓으면 Pod를 생성할 때, ConfigMap을 가져와서 컨테이너 안 환경 변수 셋팅 가능
     + Secret도 같은 역할을 하는데, 보완적인 요소의 값들을 저장하는 용도로 사용 (단, base64 인코딩을 해서 만들어야 하며, Pod로 주입될 때는 자동으로 디코딩되어 환경 변수에서는 원래의 값으로 저장)
       * 일반적인 Object 값들은 Kubernetes DB에 저장이 되지만, Secret은 메모리에 저장 (보완 측면)
       * Secret은 1MB까지만 가능하지만, 많이 사용하게 되면 시스템 자원에 영향을 미침
     + yaml 파일
       * ConfigMap을 만드는 데 이름을 지정하고, 이 데이터에 Key-Value 형태의 상수를 넣으면 됨
       * Secret도 이름을 넣고 데이터에 Key를 넣고, Value에는 값을 Base64로 변환해서 넣음
       * Pod를 생성할 떄, 컨테이너 안에 envFrom이라는 속성으로 configMapRef을 을 설정하며 이름은 cm-dev, secretRef는 이름을 sec-dev로 설정

    - 파일로 환경 변수를 설정하는 방법
      + 파일을 ConfigMap에 지정할 때는 파일 이름이 Key, 파일 안의 내용이 Value
      + 다만, 대시보드에서 지원해주지 않으므로, 직접 Master Console로 진입해 kubectl 명령을 사용 (kubectl create configmap cm-file --from-file=./file.txt : cm-file이라는 ConfigMap을 만들고, file.txt를 넣을 것이라는 명령)
      + Secret의 경우 명령어(kubectl create secret generic sec-file --from-file=./file.txt)를 사용하면 되지만, 이 명령을 통해 파일 텍스트 안의 내용이 base64로 인코딩되므로, 이미 base64로 인코딩되지 않도록 해야함
      + yaml
        * Pod를 만들어 보면, Container에 환경 변수(env)를 넣는데 이 환경 변수의 이름은 파일(file)이며, 그 파일의 값을 가져오기 위한 값은 valueFrom 속성 내 configMapKeyRef의 키를 Reference (ConfigMap의 이름은 cm-file, value는 file.txt)

     - 파일을 마운팅하여 설정하는 방법
       + Pod를 만들 때, 컨테이너 안에 Mount Path를 정의하고, 이 Path 안에 파일을 Mount
       + yaml 파일
         * volumeMounts: (컨테이너 안 볼륨을 마운트) 하는데, mountPath는 /mount이며, Mount 할 Volume을 보면 해당 볼륨 안에 ConfigMap 저장 (이름 : cm-file)

3. 파일로 환경 변수를 설정하는 방법과 파일을 마운팅하여 설정하는 방법의 차이점
   - 파일로 환경 변수를 지정하는 방법은 Pod를 생성한 다음, 각각의 ConfigMap의 내용을 변경하게 되면, Pods의 환경 변수에 영향을 미치지 못함 : 이 환경 변수 설정 방식은 한 번 주입하면 변경 불가 (Pod가 죽거나, 재생성되어야지만 변경된 값을 다시 받아와서 수정 가능)
   - 하지만, 파일을 마운트하는 방식은 원본과 연결시키는 개념으로, 그 원본이 ConfigMap이므로, 내용이 변하게 되면 Pod에 Mounting된 내용도 변경
  
