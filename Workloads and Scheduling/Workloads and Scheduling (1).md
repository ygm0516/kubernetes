### [kubernetes](https://github.com/ygm0516/kubernetes) > [ Workloads and Scheduling](https://github.com/ygm0516/kubernetes/tree/main/Workloads%20and%20Scheduling) > Workloads and Scheduling (1).md

## Workloads and Scheduling (1).md


[Kubernetes Pod](https://kubernetes.io/ko/docs/concepts/workloads/pods/) <br/>
[Kubernetes Static Pod](https://kubernetes.io/ko/docs/tasks/configure-pod-container/static-pod/) <br/>
[Kubernetes Container](https://kubernetes.io/ko/docs/concepts/containers/) <br/>
[Kubernetes Sidecar Containers](https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/) <br/>
[Kubernetes Logging Architecture](https://kubernetes.io/docs/concepts/cluster-administration/logging/) <br/>

## 목차
1. [Pod](#1)
    * [1.1. Pod 개념](#1-1)
    * [1.2. Pod 배포](#1-2)
    * [1.3. Pod 로그 라인 추출](#1-3)
2. [Static Pod](#2)
    * [2.1. Static Pod 개념](#2-1)
    * [2.2. Static Pod 생성](#2-2)
    * [2.3. Static Pod 삭제](#2-3)
3. [Multi container pod](#3)
    * [3.1. Multi container pod 개념](#3-1)
    * [3.2. Multi container pod 생성](#3-2)
4. [Streaming sidecar container](#4)
    * [4.1. Streaming sidecar container 개념](#4-1)
    * [4.2. 로그 스트리밍 사이드카 컨테이너 운영](#4-2)

# <div id='1'/> 1. Pod
## <div id='1-1'/> 1.1. Pod 개념

- 파드(Pod)
  - 쿠버네티스에서 생성하고 관리할 수 있는 배포 가능한 가장 작은 컴퓨팅 단위
  - 하나 이상의 컨테이너의 그룹
  - 이 그룹은 스토리지 및 네트워크를 공유
  - 파드는 공유 네임스페이스와 공유 파일시스템 볼륨이 있는 컨테이너들의 집합과 비슷

## <div id='1-2'/> 1.2. Pod 배포

1) kubectl 명령을 통해 아래 조건에 맞는 subtask-pod-nginx.yaml 파일을 생성 후 cat 명령을 통해 확인
> pod name: subtask-pod-nginx-yang <br/>
> image: nginx:1.14 <br/>
> port: 8080 

```
$ kubectl run subtask-pod-nginx-yang --image=nginx:1.14 --port=8080 --dry-run=client -o yaml > nginx-test-yang.yaml

$ cat nginx-test-yang.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: subtask-pod-nginx-yang
  name: subtask-pod-nginx-yang
spec:
  containers:
  - image: nginx:1.14
    name: subtask-pod-nginx-yang
    ports:
    - containerPort: 8080
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
2) subtask-yang 라는 namespace를 만들고 아래 조건에 맞는 POD를 배포하고 describe 명령을 통해 확인
> pod name: subtask-pod-01-yang<br/>
> image: busybox <br/>
> 환경변수 : CERT = "subtask-cert" <br/>
> command: /bin/sh <br/>
> args: -c "while true; do echo $(CERT); sleep 10;done"

```
$ kubectl create ns subtask-yang
namespace/subtask-yang created

$ kubectl run subtask-pod-01 --image=nginx --dry-run=client -o yaml > subtask-pod-01-yang.yaml

$ vi subtask-pod-01-yang.yaml
apiVersion: v1
kind: Pod
metadata:
  name: subtask-pod-01-yang
  namespace: subtask-yang
spec:
 containers:
 - name: subtask-pod-01-yang
   image: busybox
   env:
   - name: CERT
     value: "subtask-cert"
   command: ["/bin/sh","-c","while true; do echo $(CERT); sleep 10; done"]


$ kubectl describe pod -n subtask-yang subtask-pod-01-yang 
Name:             subtask-pod-01-yang
Namespace:        subtask-yang
Priority:         0
Service Account:  default
Node:             ta-task-cluster-1/10.200.50.84
Start Time:       Mon, 10 Feb 2025 00:32:45 +0000
Labels:           <none>
Annotations:      cni.projectcalico.org/containerID: 3d668f319414d521413944efd619616f4599bca9e9d614fd04a92a7571739592
                  cni.projectcalico.org/podIP: 10.233.104.222/32
                  cni.projectcalico.org/podIPs: 10.233.104.222/32
Status:           Running
IP:               10.233.104.222
IPs:
  IP:  10.233.104.222
Containers:
  subtask-pod-01-yang:
    Container ID:  cri-o://f43aa86b543024cce38ff24b823b0a082f62679c600fb03162878fe5157bf31c
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:79c9716db559ffde1170a4faf04910a08d930f511e6904c4899a1f7be2abfb34
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
      -c
      while true; do echo $CERT; sleep 10; done
    State:          Running
      Started:      Mon, 10 Feb 2025 00:32:48 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      CERT:  subtask-cert
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xq4wh (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-xq4wh:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason           Age   From               Message
  ----     ------           ----  ----               -------
  Warning  PolicyViolation  35s   kyverno-scan       policy restrict-seccomp-strict/check-seccomp-strict fail: validation error: Use of custom Seccomp profiles is disallowed. The fields spec.securityContext.seccompProfile.type, spec.containers[*].securityContext.seccompProfile.type, spec.initContainers[*].securityContext.seccompProfile.type, and spec.ephemeralContainers[*].securityContext.seccompProfile.type must be set to `RuntimeDefault` or `Localhost`. rule check-seccomp-strict[0] failed at path /spec/securityContext/seccompProfile/ rule check-seccomp-strict[1] failed at path /spec/containers/0/securityContext/
  Warning  PolicyViolation  65s   kyverno-admission  policy disallow-capabilities-strict/require-drop-all fail: validation failure: Containers must drop `ALL` capabilities.
  Warning  PolicyViolation  65s   kyverno-admission  policy disallow-capabilities-strict/adding-capabilities-strict pass: rule passed
  Warning  PolicyViolation  65s   kyverno-admission  policy disallow-privilege-escalation/privilege-escalation fail: validation error: Privilege escalation is disallowed. The fields spec.containers[*].securityContext.allowPrivilegeEscalation, spec.initContainers[*].securityContext.allowPrivilegeEscalation, and spec.ephemeralContainers[*].securityContext.allowPrivilegeEscalation must be set to `false`. rule privilege-escalation failed at path /spec/containers/0/securityContext/
  Warning  PolicyViolation  65s   kyverno-admission  policy restrict-seccomp-strict/check-seccomp-strict fail: validation error: Use of custom Seccomp profiles is disallowed. The fields spec.securityContext.seccompProfile.type, spec.containers[*].securityContext.seccompProfile.type, spec.initContainers[*].securityContext.seccompProfile.type, and spec.ephemeralContainers[*].securityContext.seccompProfile.type must be set to `RuntimeDefault` or `Localhost`. rule check-seccomp-strict[0] failed at path /spec/securityContext/seccompProfile/ rule check-seccomp-strict[1] failed at path /spec/containers/0/securityContext/
  Normal   Scheduled        65s   default-scheduler  Successfully assigned subtask-yang/subtask-pod-01-yang to ta-task-cluster-1
  Warning  PolicyViolation  65s   kyverno-admission  policy require-run-as-nonroot/run-as-non-root fail: validation error: Running as root is not allowed. Either the field spec.securityContext.runAsNonRoot must be set to `true`, or the fields spec.containers[*].securityContext.runAsNonRoot, spec.initContainers[*].securityContext.runAsNonRoot, and spec.ephemeralContainers[*].securityContext.runAsNonRoot must be set to `true`. rule run-as-non-root[0] failed at path /spec/securityContext/runAsNonRoot/ rule run-as-non-root[1] failed at path /spec/containers/0/securityContext/
  Warning  PolicyViolation  35s   kyverno-scan       policy disallow-capabilities-strict/require-drop-all fail: validation failure: Containers must drop `ALL` capabilities.
  Warning  PolicyViolation  35s   kyverno-scan       policy require-run-as-nonroot/run-as-non-root fail: validation error: Running as root is not allowed. Either the field spec.securityContext.runAsNonRoot must be set to `true`, or the fields spec.containers[*].securityContext.runAsNonRoot, spec.initContainers[*].securityContext.runAsNonRoot, and spec.ephemeralContainers[*].securityContext.runAsNonRoot must be set to `true`. rule run-as-non-root[0] failed at path /spec/securityContext/runAsNonRoot/ rule run-as-non-root[1] failed at path /spec/containers/0/securityContext/
  Warning  PolicyViolation  35s   kyverno-scan       policy disallow-privilege-escalation/privilege-escalation fail: validation error: Privilege escalation is disallowed. The fields spec.containers[*].securityContext.allowPrivilegeEscalation, spec.initContainers[*].securityContext.allowPrivilegeEscalation, and spec.ephemeralContainers[*].securityContext.allowPrivilegeEscalation must be set to `false`. rule privilege-escalation failed at path /spec/containers/0/securityContext/
  Normal   Pulling          66s   kubelet            Pulling image "busybox"
  Normal   Started          63s   kubelet            Started container subtask-pod-01-yang
  Normal   Created          63s   kubelet            Created container subtask-pod-01-yang
  Normal   Pulled           63s   kubelet            Successfully pulled image "busybox" in 2.314s (2.314s including waiting). Image size: 4517392 bytes.

```





## <div id='1-3'/> 1.3. Pod 로그 라인 추출
```
$ kubectl logs dns-autoscaler-6ffb84bd6-jqwmm -n kube-system | grep "dns" > find-dns-autoscaler-yang.log
$ cat find-dns-autoscaler-yang.log
I0207 00:17:51.362607       1 autoscaler.go:49] Scaling Namespace: kube-system, Target: deployment/coredns
I0207 00:17:51.616962       1 autoscaler_server.go:157] ConfigMap not found: configmaps "dns-autoscaler" not found, will create one with default params
I0207 00:17:51.626278       1 k8sclient.go:148] Created ConfigMap dns-autoscaler in namespace kube-system
I0207 00:20:21.367770       1 autoscaler_server.go:157] ConfigMap not found: Get "https://10.233.0.1:443/api/v1/namespaces/kube-system/configmaps/dns-autoscaler": dial tcp 10.233.0.1:443: connect: connection refused, will create one with default params
I0207 00:20:31.364314       1 autoscaler_server.go:157] ConfigMap not found: Get "https://10.233.0.1:443/api/v1/namespaces/kube-system/configmaps/dns-autoscaler": dial tcp 10.233.0.1:443: connect: connection refused, will create one with default params
I0207 00:24:31.367253       1 autoscaler_server.go:157] ConfigMap not found: Get "https://10.233.0.1:443/api/v1/namespaces/kube-system/configmaps/dns-autoscaler": dial tcp 10.233.0.1:443: connect: connection refused, will create one with default params
I0207 00:24:41.380260       1 autoscaler_server.go:157] ConfigMap not found: Get "https://10.233.0.1:443/api/v1/namespaces/kube-system/configmaps/dns-autoscaler": dial tcp 10.233.0.1:443: connect: connection refused, will create one with default params
I0207 00:24:51.365231       1 autoscaler_server.go:157] ConfigMap not found: Get "https://10.233.0.1:443/api/v1/namespaces/kube-system/configmaps/dns-autoscaler": dial tcp 10.233.0.1:443: connect: connection refused, will create one with default params
I0207 00:25:01.364088       1 autoscaler_server.go:157] ConfigMap not found: Get "https://10.233.0.1:443/api/v1/namespaces/kube-system/configmaps/dns-autoscaler": dial tcp 10.233.0.1:443: connect: connection refused, will create one with default params
I0207 00:25:11.364799       1 autoscaler_server.go:157] ConfigMap not found: Get "https://10.233.0.1:443/api/v1/namespaces/kube-system/configmaps/dns-autoscaler": dial tcp 10.233.0.1:443: connect: connection refused, will create one with default params
```

# <div id='2'/> 2. Static Pod
## <div id='2-1'/>2.1. Static Pod란.
- Static Pod 개념
  - API 서버 없이 특정 노드에 있는 kubelet 데몬에 의해 직접 관리되어지는 파드
  - 컨트롤 플레인에 의해 관리되는 파드와는 달리, kubelet 이 각각의 스태틱 파드를 감시
  - 스태틱 파드는 항상 특정 노드에 있는 하나의 Kubelet에 매여 있음
  - 노드에서 구동되는 파드는 API 서버에 의해서 볼 수 있지만, API 서버에서 제어될 수는 없음
  - 파드 이름에는 노드 호스트 이름 앞에 하이픈을 붙여 접미사로 추가됨
- Static Pod 경로 변경
  - kubelet의 config 파일 (/var/lib/kubelet/config.yaml) staticPodPath: /etc/kubernetes/manifests가 정의되어 있음
    - 디렉토리 수정 시 kubelet 데몬 재실행
```
$ systemctl restart kubelet
```
## <div id='2-2'/>2.2. Static Pod 생성
- kubectl 명령어는 master node에서만 실행가능
```
$ kubectl run subtask-static-pod-nginx-yang --image=nginx --port=80 --dry-run=client -o yaml > subtask-static-pod-nginx-yang.yaml
$ cat subtask-static-pod-nginx-yang.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: subtask-static-pod-nginx-yang
  name: subtask-static-pod-nginx-yang
spec:
  containers:
  - image: nginx
    name: subtask-static-pod-nginx-yang
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
- 위에서 생성한 subtask-static-pod-nginx-yang.yaml 파일을 배포할 노드 /etc/kubernetes/manifests/에 생성
```
$ ssh [배포할 노드명]
$ cd /etc/kubernetes/manifests/
$ sudo su
# vi subtask-static-pod-nginx-yang.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: subtask-static-pod-nginx-yang
  name: subtask-static-pod-nginx-yang
spec:
  containers:
  - image: nginx
    name: subtask-static-pod-nginx-yang
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

# ls
nginx-proxy.yml  subtask-static-pod-nginx-yang.yaml

# exit

$ kubectl get pod
NAME                                              READY   STATUS    RESTARTS     AGE
nfs-pod-provisioner-668f59bd7f-747gt              1/1     Running   4 (3d ago)   3d
subtask-static-pod-nginx-yang-ta-task-cluster-2   1/1     Running   0            50s

```
## <div id='2-3'/>2.3. Static Pod 삭제
- kubectl 명령어는 master node에서 실행
```
$ ssh [배포한 노드명]

$ cd /etc/kubernetes/manifests/

$ sudo su

# rm subtask-static-pod-nginx-yang.yaml

$ kubectl get pod
NAME                                   READY   STATUS    RESTARTS     AGE
nfs-pod-provisioner-668f59bd7f-747gt   1/1     Running   4 (3d ago)   3d
```

# <div id='3'/> 3.Multi container pod
## <div id='3-1'/>3.1. Container와 pod의 차이점
- Pod <br/>
쿠버네티스에서 생성하고 관리할 수 있는 배포 가능한 가장 작은 컴퓨팅 단위(컨테이너를 표현하는 K8S API의 최소 단위) <br/>
하나 이상의 컨테이너 그룹(여러 개의 컨테이너가 포함될 수 있음)

- Container <br/>
어플리케이션을 의미함 

* kubernetes - conatiner 공식 문서 <br/>
쿠버네티스 클러스터에 있는 개별 node는 해당 노드에 할당된 파드를 구성하는 컨테이너들을 실행한다.  <br/>
파드 내부에 컨테이너들은 같은 노드에서 실행될 수 있도록 같은 곳에 위치하고 함께 스케줄된다

## <div id='3-2'/>3.2. Multi container pod 생성
```
$ kubectl run subtask-multi-pod-yang --image=nginx --dry-run=client -o yaml > subtask-multi-pod-yang.yaml
$ vi subtask-multi-pod-yang.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: subtask-multi-pod-yang
  name: subtask-multi-pod-yang
  namespace: subtask-yang
spec:
  containers:
  - image: nginx
    name: nginx
  - image: redis //추가
    name: redis
  - image: memcached //추가
    name: memcached
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

$ kubectl get pod -n subtask-yang 
NAME                     READY   STATUS    RESTARTS   AGE
subtask-multi-pod-yang   3/3     Running   0          28s
```

# <div id='4'/> 4. Streaming sidecar container
## <div id='4-1'/>4.1. Streaming sidecar container 개념
- sidecar container
  - 동일한 애플리케이션 컨테이너와 함께 실행되는 보조 컨테이너 
  - 기본 애플리케이션 코드를 직접 변경하지 않고 로깅, 모니터링, 보안 또는 데이터 동기화와 같은 추가 서비스나 기능을 제공하여 기본 앱 컨테이너 의 기능을 향상 또는 확장하는 데 사용
  - 본래의 컨테이너는 기본 서비스에 충실하고, 추가 기능을 별도의 컨테이너에 적용

## <div id='4-2'/>4.2. 로그 스트리밍 사이드카 컨테이너 운영
```
$ vi sidecar-yang.yaml
apiVersion: v1
kind: Pod
metadata:
  name: counter-yang
  namespace: subtask-yang
spec:
  containers:
  - name: count-yang
    image: busybox:1.28
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/1.log;
        echo "$(date) INFO $i" >> /var/log/2.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: varlog-yang
      mountPath: /var/log
  - name: count-log-1-yang
    image: busybox:1.28
    command: ["/bin/sh", "-c", "tail -n+1 -F /var/log/1.log"]
    volumeMounts:
    - name: varlog-yang
      mountPath: /var/log
  - name: count-log-2-yang
    image: busybox:1.28
    command: ["/bin/sh", "-c", "tail -n+1 -F /var/log/2.log"]
    volumeMounts:
    - name: varlog-yang
      mountPath: /var/log
  volumes:
  - name: varlog-yang
    emptyDir: {}

$ kubectl apply -f sidecar-yang.yaml 
$ kubectl get pod -n subtask-yang
NAME                     READY   STATUS    RESTARTS   AGE
counter-yang             3/3     Running   0          89m


$ kubectl logs -n subtask-yang counter-yang -c count-log-1-yang --tail=5
5760: Mon Feb 10 04:05:00 UTC 2025
5761: Mon Feb 10 04:05:01 UTC 2025
5762: Mon Feb 10 04:05:03 UTC 2025
5763: Mon Feb 10 04:05:04 UTC 2025
5764: Mon Feb 10 04:05:05 UTC 2025

$ kubectl logs -n subtask-yang counter-yang -c count-log-2-yang --tail=5
Mon Feb 10 04:05:19 UTC 2025 INFO 5778
Mon Feb 10 04:05:20 UTC 2025 INFO 5779
Mon Feb 10 04:05:21 UTC 2025 INFO 5780
Mon Feb 10 04:05:22 UTC 2025 INFO 5781
Mon Feb 10 04:05:23 UTC 2025 INFO 5782


-- 실시간 확인
$ kubectl logs -f -n subtask-yang counter-yang -c count-log-1-yang
$ kubectl logs -f -n subtask-yang counter-yang -c count-log-2-yang
```










