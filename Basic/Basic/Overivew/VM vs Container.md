-----
### VM vs Container
-----
1. 시스템 구조의 차이
<div align="center">
<img src="https://github.com/user-attachments/assets/05d33a90-2986-4a45-8f62-e3ee07c037d4">
</div>

   - 공통적으로 하나의 서버가 존재(Host Server), 그리고 한 서버에는 운영체제가 Host OS로 적재
   - VM의 경우, 이 Host OS 위에 VM을 가상화시켜주기 위해 여러 Hypervisor가 존재하며, 이를 통해 원하는 운영체제를 Guest OS로 적재해 여러 VM을 만들 수 있음
     + Guest OS도 Host OS와 동일하게, 하나의 OS를 독립적으로 가지고 있는 것처럼 사용 가능
     + 여러가지 애플리케이션을 실행하고, 각 서비스를 실행 가능

   - Container의 경우, Host OS 설치까지는 동일하지만, 이 OS에 컨테이너 가상화 시켜주는 소프트웨어를 통해 Container를 생성
<div align="center">
<img src="https://github.com/user-attachments/assets/c42ae017-55d4-4444-a02e-6d06ce86923f">
</div>

   - Container
     + Linux 마다 Version이 존재 : 이 버전에 따라 기본적 설치 라이브러리가 다르므로, 버전 차이에 따른 문제 발생
     + 따라서 Docker와 같은 Container를 설치하고 이를 통해 컨테이너 이미지를 만들 수 있는데, 이 이미지에는 한 서비스와 그 서비스에 돌아가는 필요한 라이브러리들이 존재
     + 또한, 여러 컨테이너들 간의 호스트 자원을 분리해서 쓰도록 해줌 : Liunx 고유 기술인 namespace와 cgroups를 사용해서 격리
       * namespace : 커널에 관련된 영역 분리
       * cgroups : 자원에 대한 영역 분리
     + 즉, Docker와 같이 Container 가상화 솔루션들은 OS에서 제공하는 자원 격리 기술을 이용해 Container라는 단위로 서비스를 분리할 수 있게 해줌
     + 이를 사용하면, 컨테이너 가상화가 설치된 OS에는 개발환경에 대한 걱정없이 배포 가능
<div align="center">
<img src="https://github.com/user-attachments/assets/fa675a75-fe7c-49a9-9401-f7d20a9963cf">
</div>

2. VM와 Container의 단점
  - VM : 새로운 Guest OS를 설치할 때, LinuxOS를 설치해서 사용 가능하며, 보안적으로 Guest OS와 Host OS가 분리되어 있으므로 VM 끼리 서로 피해가 가지 않음 
  - Container : Linux OS에서 윈도우용 컨테이너를 사용 불가하며, 보안적으로 취약함 (Host OS가 보약에 취약해지면, 다른 컨테이너들도 취약해짐)

3. 개발 사상적 차이점
<div align="center">
<img src="https://github.com/user-attachments/assets/bbe75ee1-4d1e-4c8a-9238-3e5c07dd48f4">
</div>

  - 일반적으로 서비스를 만들 때, 한 가지 언어를 사용해서 여러 모듈들이 한 서비스로 같이 돌아감
    + 따라서, 만약 A, B 모듈은 괜찮지만, C 모듈에 부하가 많이 가는 상황이라면, VM을 하나 더 생성
    + 자원, 성능 입장에서 Guest OS는 2개 (A, B는 확장할 필요가 없지만, 한 패키지이므로 그대로 적재)

  - 컨테이너의 경우, 마이크로 서비스(MS), 즉, 한 서비스를 만들 때, 모듈별로 나누어 각각의 컨테이너에 담는 걸 권장
    + 그리고 그 모듈에 맞는 최적화된 개발 언어를 사용하도록 권장

  - Kubernetes는 여러 컨테이너들을 한 Pod(파드)라는 개념에 묶을 수 있으며, 한 Pod에 한 Container에 담을 수 있음 : 따라서, 한 Pod가 하나의 배포 단위이므로, 필요한 Pod만 확장 가능
