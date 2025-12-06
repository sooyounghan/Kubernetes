-----
### Configmap, Secret
-----
<div align="center">
<img width="1569" height="666" alt="image" src="https://github.com/user-attachments/assets/e00d1565-000f-4fb2-9d81-6871a1441609" />
</div>

1. Pod에 바로 연결이 되며, Object의 속성을 보면 데이터를 담을 수 있으며, 사용자가 데이터를 넣고 Pod에 값을 주입 가능
   - envFrom : Pod 안에 환경 변수로 들어가게 하는 속성으로, ConfigMap을 연결하고 Pod 안에 env 명령을 입력하며 Configmap에 데이터가 주입된 것 확인 가능
   - volumes : Pod와 특정 저장소를 연결하는 속성이며, Secrt을 연결하고 Pod 안에 들어가서 Mounting된 Path를 조회하면 Secret에 stringData가 존재

2. Configmap의 data
   - Key-Value 형식
     + spring_profiles_active : 인프라에는 다양한 환경, 즉, 검증 / 운영 환경 등 존재하는데, App이 어느 환경에서 기동되는지 알려주기 위한 변수
     + application_role : App의 역할을 지정
     + postgresql_filepath : Secret Data로 연결할 파일의 경로 (Pod의 mountPath에서 지정) - 환경변수로 주면, 기동할 때 환경 변수 값을 보고 DB 정보를 확인해 접속할 수 있도록 구성되어 있으므로 Pod에서 mountPath 경로를 바꾸고 싶을때, App을 다시 빌드하지 않아도 Configmap만 수정해서 간단하게 처리 가능

   - 즉, 크게 인프라 환경에 따른 값이 있고, App의 기능을 제어하기 위한 값과 외부 환경을 App으로 주입시키기 위한 값들이 존재

3. Pod가 생성되면, Configmap에 있는 데이터의 모든 내용이 컨테이너 내부 환경 변수로 주입
   - 이미지를 만들 때, java 실행 파일 명령을 넣어놨는데, 이 명령이 컨테이너 생성 후 자동으로 동작하면서 환경 변수 값들이 매칭이 됨 (만약, spring_profiles_active가 없다면 이 값은 null)

4. Secret
   - Database 정보가 존재
   - postrgresql-info.yaml 존재
   - Object를 저장하면, stringData는 쓰기 전용 속성이고, 실제 저장은 Configmap과 같이 data라는 속성에 저장이 되는데, Key은 그대로 있지만, Value에 해당하는 부분이 base64로 인코딩된 값으로 변경되어 저장

5. Pod에 mountPath라는 속성을 주면, Container 안에 path가 생성되며, Volume과 매칭이 되어서 secret과 연결이 되고, 컨테이너 안에 파일이 생성
   - 이 때, Value 값이 다시 Decoding 되어 원래 입력한 형태의 값으로 설정
   - App이 기동할 때, 이 파일을 읽어서 DB에 연결하도록 로직 처리

6. DB 정보를 Configmap에 설정해 Volume 형태로 연결해도 되며, 반대로 Secret을 envFrom으로 환경 변수로 넣을 수 있음 : 기능적으로 가능하지만, 최대한 지양하는 것이 좋음

-----
### 동작 확인
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/8597087f-c799-4ffd-95e3-831d8953e80c" />
</div>

1. 입력값 확인 (Secret, Configmap)
<div align="center">
<img src="https://github.com/user-attachments/assets/4e66854f-5011-427d-8383-e73ae0b4aec2" />
<img src="https://github.com/user-attachments/assets/0c107fa5-d968-445e-afa9-5b4820d82ea7" />
</div>

   - kubectl
```bash
// Configmap 확인
kubectl describe -n anotherclass-123 configmaps api-tester-1231-properties
kubectl get -n anotherclass-123 configmaps api-tester-1231-properties -o yaml
kubectl get -n anotherclass-123 configmaps api-tester-1231-properties -o jsonpath='{.data}'

// Secret 확인
kubectl get -n anotherclass-123 secret api-tester-1231-postgresql -o yaml
kubectl get -n anotherclass-123 secret api-tester-1231-postgresql -o jsonpath='{.data}'

// Secret data에서 postgresql-info가 Key인 Value값만 조회 하기
kubectl get -n anotherclass-123 secret api-tester-1231-postgresql -o jsonpath='{.data.postgresql-info\.yaml}'

// Secret data에서 postgresql-info가 Key인 Value값을 Base64 디코딩해서 보기
kubectl get -n anotherclass-123 secret api-tester-1231-postgresql -o jsonpath='{.data.postgresql-info\.yaml}' | base64 -d
```

2. 컨테이너 내부 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/e92ff2b8-da44-464d-9351-5a6f8775d4c2" />
</div>

  - Dashboard
```bash
// Configmap 환경 변수 확인
env

// Secret 파일 확인
ls /usr/src/myapp/datasource
cat /usr/src/myapp/datasource/postgresql-info.yaml

// java 실행 인자 확인
jps -v
```

   - kubectl
```bash
// 사용 포맷
kubectl exec -n <namespace-name> -it <pod-name> -- <command>

kubectl exec -n anotherclass-123 -it api-tester-1231-5c9f87b99c-4bdx6 -- env
kubectl exec -n anotherclass-123 -it api-tester-1231-5c9f87b99c-4bdx6 -- cat /usr/src/myapp/datasource/postgresql-info.yaml
kubectl exec -n anotherclass-123 -it api-tester-1231-5c9f87b99c-4bdx6 -- jps -v
```

3. API 확인
```bash
﻿// Application 정보 확인 (version, profile, role, database) 
http://192.168.56.30:31231/info 

// Application Properties 파일 구성 확인
http://192.168.56.30:31231/properties
```

4. Configmap 수정
<div align="center">
<img src="https://github.com/user-attachments/assets/2424a09f-fa95-47d7-99ce-e76c5e0b079b" />
</div>

```bash
﻿// 환경변수 수정 명령
export application_role=GET
```

5. API 재확인 & Pod 재기동 후 확인
