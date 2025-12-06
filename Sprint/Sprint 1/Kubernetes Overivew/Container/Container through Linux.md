-----
### 리눅스 흐름으로 이해하는 컨테이너
-----
1. 기술의 흐름으로 이해하는 컨테이너
<div align="center">
<img src="https://github.com/user-attachments/assets/7669b96b-0c9f-40fe-9937-5a974c83df8b">
</div>

2. 리눅스 흐름으로 이해하는 컨테이너
   - 최초의 OS인 UNIX가 등장하였고, 유료화 문제로 Linux가 등장
     + debian Linux : 커뮤니티 용으로 무료
       * Ubuntu : debian 기반으로 만든 배포판으로, debian에 비해 사용자 편의 기능이 더 많으며, Linux에서 최초로 Windowns 처럼 UI를 제공 (학습용이나 클라우드 환경)  
     + redhat Linux : 유료 (기업에서 많이 사용)
       * 최초 Fedora Linux라고 하는 새로운 기능을 개발한 버전 (무료)
       * 이 기능이 안정화되면서 Redhat Linux로 이름이 바뀌어서 Release (유료 버전)
       * RedHat Enterprise Linux, RHEL를 복제해서 만든 것이 CentOS (RHEL과 똑같은 안정화 버전인데, 무료로 사용 가능)
         * 일반적으로 개발 환경에서는 CentOS, 운영 환경에서 RHEL를 사용하기도 함
         * 하지만, CentOS 8은 2021년에 지원 종료, CentOS 7은 2024년에 지원 종료 (Ubuntu의 시장 점유율이 압도적)
       * 2019년 IBM에 인수가 되면서, Fedora를 통해 기능 개발을 하며, CentOS Stream이라는 기능을 테스트 배포판(테스트베드)으로 사용 (이 배포판에서는 바이너리 호환성 보장이 안될 수 있음)
       * 테스트 과정이 끝나면 안정화 버전인 Redhat Linux가 됨 

   - CentOS 지원 종료에 대한 대처 방법
     + Redhat Linux로 전환해서 유료로 사용 : CentOS가 RHEL의 복사본이므로 Migration 하는 게 기존 App에게 안전한 방법 (IBM 권고)
     + CentOS를 그대로 기술 지원을 사용하는 방법
     + 타 Linux OS로 Migration Script 제공
     + Redhat Linux를 복제해 기존 CentOS와 같이 무료 배포판을 만드는 프로젝트가 존재 : Rocky Linux, AlmaLinux

   - 정리
     + 주로 사용하는 리눅스에는 Debian과 Redhat 계열이 존재
     + Kubernetes 설치도 크게 이 두가지 계열을 지원
     + Redhat 계열의 하나인 CentOS는 종료되었음
     + Rocky Lunux는 CentOS 대체제 중 하나
     + 결론 : Rocky Lunux는 Kubernetes를 설치할 때, Redhat 계열로 보면 됨

<div align="center">
<img src="https://github.com/user-attachments/assets/3a42295a-25ed-45f7-8531-7ff3001bc59b">
</div>

3. 컨테이너 (Container)
   - chroot : 사용자 격리를 시작으로 파일이나 네트워크를 분리하는 기술
   - cgroup : 각 Application마다 CPU나 Memory를 할당할 수 있도록 자원 격리
   - namespace : 보통 Application 하나가 하나의 프로세스를 차지하는데, 이 프로세스를 격리시켜주는 기술
   - 따라서, 독립적인 환경에서 실행할 수 있도록 도와줌
   - 이 기술들을 집약해서 정리한 것 : LinuX Container(LXC)로, 최초 Container
     + 이 기술을 기반으로 만들어진 것이 Docker (Docker는 보안적인 부분에 약함 : root 권한으로 설치하고 실행해야 하므로 rootless 기능이 추후 등장)
     + 이후 rkt(rocket)이라는 컨테이너 등장 (Docker의 보안적 부분을 해결한 컨테이너)
        * rkt는 컨테이너 전용 Linux라고 해서 CoreOS가 있고, 이 OS에 대한 대표 런타임이 있는데, Redhat에 인수됨녀서 Fedora CoreOS로 이름이 변경
        * Redhat에서는 cri-o(크라이오)라는 컨테이너를 개발 중
     + 이후, Containerd와 cri-o가 등장
        * Containerd : Docker에 컨테이너를 만들어주는 기능이 분리된 것 
4. 컨테이너 오케스트레이션 (Container Orchestration)
<div align="center">
<img src="https://github.com/user-attachments/assets/316b7ea0-7ead-443e-a1c4-e001e748928e">
</div>

   - 컨테이너 (Container)
     + 사용자가 Docker에게 Application을 생성해달라고 명령을 전송
     + Docker는 컨테이너를 생성
     + 만들어진 컨테이너가 버전 1의 Application
     + 현재 Application은 가동 중이며, 트래픽도 계속 전송 중이며, 서비스가 운영 중인 상태
     + 버전 2를 생성하지만, 가동 시간까지 시간이 소모되며, 컨테이너를 조회하는 기동 확인 과정이 필요
       * 기동이 완료되면, V1으로 흘러가던 사용자 트래픽을 App V2로 전환시켜줘야 하기 위해, Docker는 네트워크 수정 명령이 발생
       * 네트워크 수정이 완료되면 App V1 컨테이너를 삭제
      + 즉, 단일 Application을 가동할 때는 위와 같은 방법으로 업그레이드를 실시 
     
   - 컨테이너 오케스트레이션 (Container Orchestration)
     + 사용자가 Kubernetes에게 Application 생성 명령을 전송
     + Kubernetes가 Docker에게 한 번 더 전달하는 과정이 존재
     + Docker를 통해 컨테이너 생성
     + Kubernetes에게 App V2로 업그레이드 하라는 명령 전송
       * Kubernetes는 App V2 컨테이너 생성
       * Kubernetes가 App V2를 모니터링하며, 컨테이너가 기동 완료되면 네트워크를 수정해 트래픽을 전환
       * 또한, App V1도 자동 삭제

   - 즉, App을 컨테이너에 담아서 배포하며, 시스템의 운영 노하우를 많이 가지고 있는 것
   - Docker 이후 Kubernetes가 등장하였고, 여러 가지 오케스트레이션이 등장

5. 정리
   - Kubernetes는 현재 여러 분야에서 활용되고 있음
   - 컨테이너를 더 쉽게 사용할 수 있게 해줌
   - 컨테이너는 Kubernetes와 인터페이스가 중요함
