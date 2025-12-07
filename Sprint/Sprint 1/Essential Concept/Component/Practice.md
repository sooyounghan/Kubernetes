-----
### 실습 - Component 동작으로 이해하기 
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/6804f55e-d3ca-44d6-8c19-154a405671a2" />
</div>

1. 주요 컴포넌트 구성
   - Resource 확인
```bash
kubectl api-resources
```

  - Cluster 주요 컴포넌트 로그 확인
```bash
// 주요 컴포넌트 로그 보기 (kube-system)
kubectl get pods -n kube-system
kubectl logs -n kube-system etcd-k8s-master
kubectl logs -n kube-system kube-scheduler-k8s-master
kubectl logs -n kube-system kube-apiserver-k8s-master
...
```

  - Master Node 파일 위치
```bash
// 쿠버네티스 인증서 위치
cd /etc/kubernetes
ls /root/.kube/config

// Control Plane Component Pod 생성 yaml 파일 위치
ls /etc/kubernetes/manifests

// 전체 Pod 로그
/var/log/pods/<namespace_<pod-name>_<uid>/<number>.log
/var/log/containers/<pod-name>_<namespace>_<container-name>_<container-id>.log
```

  - 트러블 슈팅
```bash
// kubelet 상태 확인
1) systemctl status kubelet       // systemctl (restart or start) kubelet
2) journalctl -u kubelet | tail -10

// 상태 확인 -> 상세 로그 확인 -> 10분 구글링 -> VM 재기동 -> Cluster 재설치 ->  답을 찾을 때 까지 구글링

// containerd 상태 확인
1) systemctl status containerd
2) journalctl -u containerd | tail -10

// 노드 상태 확인
1) kubectl get nodes -o wide
2) kubectl describe node k8s-master

// Pod 상태 확인
1) kubectl get pods -A -o wide

// Event 확인 (기본값: 1h)
2-1) kubectl get events -A
2-2) kubectl events -n anotherclass-123 --types=Warning  (or Normal)

// Log 확인
3-1) kubectl logs -n anotherclass-123 <pod-name> --tail 10    // 10줄 만 조회하기
3-2) kubectl logs -n anotherclass-123 <pod-name> -f           // 실시간으로 조회 걸어 놓기
3-3) kubectl logs -n anotherclass-123 <pod-name> --since=1m   // 1분 이내에 생성된 로그만 보기
```

2. Service 동작
```bash
iptables -t nat -L KUBE-NODEPORTS -n  | column -t
```
  - Calico 버전이 업그레이드 되면서 iptables에 nodePort 정보를 표시해 주고 있지 않음
  - nodePort 정보는 kubectl get svc -A를 통해서만 확인 가능
    
```bash
[root@k8s-master ~]# iptables -t nat -L KUBE-NODEPORTS -n  | column -t
Chain                      KUBE-NODEPORTS  (1   references)
target                     prot            opt  source       destination
KUBE-EXT-AWA2CQSXVI7X2GE5  tcp             --   0.0.0.0/0    0.0.0.0/0    /*  monitoring/grafana:http                    */
KUBE-EXT-CEZPIJSAUFW5MYPQ  tcp             --   0.0.0.0/0    0.0.0.0/0    /*  kubernetes-dashboard/kubernetes-dashboard  */
```
