-----
### ConfigMap, Secret - Env(Literal, File), Mount(File)
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/970fd838-9993-4772-a296-749e25b6124f">
</div>

1. Env (Literal)
  - Pod 생성시 환경 변수를 주입하기 위해 사용
  - Configmap, Secret의 내용이 수정되면 Pod를 재생성해야 다시 반영됨
  - Key:Value 형식으로 입력, key와 value는 모두 String이기 때문에 true, false의 경우 'true'로 입력
  - Secret의 경우 value는 Base64 인코딩이 필수, 추후 Pod 내부 env로 들어갈 때는 디코딩이 됨
  - Secret은 내용을 메모리영역에 올려놓고 사용한다는 점에서 Configmap보단 보안에 유리하게 기능이 동작함
  - 한 ConfigMap과 Secret의 파일 크기는 1Mbyte를 넘을 수 없음 (Etcd에서 1.5Mib로 제한이 있음)
  - ConfigMap (kubectl create configmap cm-file --from-literal=key1=value1 --from-literal=key2=value2)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-dev
data:
  SSH: 'false' # 모두 String 값이므로, boolean 값을 넣고 싶다면 ''으로 설정 필요
  User: dev
```
  - cm-dev
```json
{
	"SSH": "false",
	"User": "dev"
}
```

   - Secret : kubectl create secret generic sec-file --from-literal=key1=value1
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: sec-dev
data:
  Key: MTIzNA== # base64 인코딩 형태의 문자 입력
```
  - sec-dev
<div align="center">
<img src="https://github.com/user-attachments/assets/8c20aa9e-cf16-4e74-9e87-505dc9730bc8">
</div>

  - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: container
    image: kubetm/init
    envFrom:
    - configMapRef:
        name: cm-dev
    - secretRef:
        name: sec-dev
```

  - Pod 내부로 들어가서 주입 된 Configmap과 Secret 확인
```bash
env
```
```bash
[root@pod-1 /]# env
...
SSH=false
...
Key=1234 (디코딩 되어 저장)
...
User=dev
...
```

2. Env (File)
  - Configmap의 key는 file-c.txt가 되지만, 환경변수로 들어갔을 경우 .txt는 허용되지 않기 때문에 제거
  - Configmap의 key를 file로 만들고, value를 file-c.txt 안의 내용으로 넣고 싶을 경우 명령어 : kubectl create configmap cm-file --from-file=file=./file-c.txt
  - Configmap : Master Node에서 명령어 입력
```bash
echo "Content" >> file-c.txt
kubectl create configmap cm-file --from-file=./file-c.txt
```

  - Secret (Master Node에서 명령어 입력)
```bsh
echo "Content" >> file-s.txt
kubectl create secret generic sec-file --from-file=./file-s.txt
```
   - 명령어로 Secret를 생성할 때는 내용을 자동으로 Base64 해주기 때문에, 별도로 "Content"를 Base64 해줄 필요 없음 
<div align="center">
<img src="https://github.com/user-attachments/assets/41b39818-a95a-4083-b044-d1305ecda055">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/ace5c54c-9abd-4dc1-bed1-686198bfa71b">
</div>

   - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-file
spec:
  containers:
  - name: container
    image: kubetm/init
    env:
    - name: file-c
      valueFrom:
        configMapKeyRef:
          name: cm-file
          key: file-c.txt
    - name: file-s
      valueFrom:
        secretKeyRef:
          name: sec-file
          key: file-s.txt
```
   - Pod 내부로 들어가서 주입 된 Configmap과 Secret 확인
```bash
env
```
```bash
[root@pod-file /]# env
file-c=Content
...
file-s=Content
...
PWD=/
```

3. Volume Mount (File)
  - 마운팅으로 연결하기 때문에 Configmap이나 Secret를 수정시 사용된 Pod의 환경 변수도 즉시 변경됨
  - Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-mount
spec:
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: file-volume
      mountPath: /mount # 파일을 넣을 Path
  volumes:
  - name: file-volume
    configMap:
      name: cm-file # 해당 파일을 ConfigMap으로 연결
```

  - Pod 내부로 들어가서 마운팅 된 Configmap 확인
```bash
cd /mount
ls
cat file-c.txt
```
```bash
[root@pod-mount /]# ls  
anaconda-post.log  bin  boot  dev  etc  home  lib  lib64  media  mnt  mount  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@pod-mount /]# cd mount
[root@pod-mount mount]# ls
file-c.txt
```

4. ConfigMap 편집
```json
{
	"file-c.txt": "Content123
		"
}
```
   - 현재 pod-file 내용은 변경되지 않지만, 해당 pod 삭제 후 재생성하면 값이 변경
```bash
[root@pod-file /]# env
file-c=Content123
```

  - 현재 pod-mount의 경우에는 바로 변경
```bash
[root@pod-mount /]# env
file-c=Content123
```

  - 실습 후 모든 리소스 삭제 (Dashboard에서 리소스별 삭제 or Master Node에서 아래 명령 실행)
```bash
kubectl delete pod pod-1 pod-file pod-mount
kubectl delete cm cm-dev cm-file
kubectl delete secret sec-dev sec-fil
```
