-----
### Kustomize 패키지 구성 요소
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/8f82d53e-4aa4-4027-9741-29a16fe07a35" />
</div>

1. 직접 폴더를 생성 및 하위 폴더 구성도 직접 생성해야 하며, 구조에 배포할 YAML 파일도 직접 생성
2. api-tester : 메인 폴더
   - base : Default Format 될 YAML 파일을 넣는 폴더 (즉, Template 폴더와 Mapping되는 폴더)
     + deployment.yaml : 베이스 YAML 파일
     + kustomization.yaml : 배포할 파일 선택 및 공통값 설정 YAML 파일로 Resource가 있어 배포할 파일들만 명시적으로 작성하면 됨
       * commonLabels라는 키가 있어서 내용을 넣으면 배포될 모든 YAML 파일에 commonLabels 내용이 들어감

   - overlays : 오버레이 할 영역
     + 배포할 때 폴더별로 선택 가능 : 해당 폴더마다 그 환경에 맞게 Overlay 될 파일들을 넣는 것
     + kustomization.yaml : 위와 역할 동일 (이 파일들에 대해 배포할 파일을 지정하고 공통값 설정 가능) / helm의 경우 values.yaml파일에
     + deployment.yaml : Overlay될 파일 (kubectl apply -k ./app/overlays/dev)
