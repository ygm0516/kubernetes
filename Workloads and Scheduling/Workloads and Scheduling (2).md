### [kubernetes](https://github.com/ygm0516/kubernetes) > [Workloads and Scheduling](https://github.com/ygm0516/kubernetes/tree/main/Workloads%20and%20Scheduling) > Workloads and Scheduling (2).md

## Workloads and Scheduling (2).md


[Kubernetes Deployment](https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/) <br/>
[Scale-in/Scale-out 명령어](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#scale) <br/>
[Updating a Deployment ](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment) <br/>
[Node drain](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_drain/) <br/>
[Pod Scheduling](https://kubernetes.io/ko/docs/concepts/scheduling-eviction/assign-pod-node/) <br/>
[ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/) <br/>
[Secret](https://kubernetes.io/docs/concepts/configuration/secret/) <br/>

## 목차
- [ 1. Deployment](#-1-deployment)
  - [ 1.1. Deployment, Scale-in/Scale-out 개념](#-11-deployment-scale-inscale-out-개념)
  - [ 1.2. deployment pod 확장](#-12-deployment-pod-확장)
- [ 2. Rolling Update \& Roll back](#-2-rolling-update--roll-back)
  - [ 2.1. Rolling Update / Roll back 개념](#-21-rolling-update--roll-back-개념)
  - [ 2.2. Rolling Update / Roll back 수행 명령어](#-22-rolling-update--roll-back-수행-명령어)
  - [ 2.3. Rolling Update / Roll back 실습](#-23-rolling-update--roll-back-실습)
- [ 3. Node management](#-3-node-management)
  - [ 3.1. kubectl drain option 개념](#-31-kubectl-drain-option-개념)
  - [ 3.2. 스케줄링 불가능 설정](#-32-스케줄링-불가능-설정)
  - [ 3.3. Ready 상태인 node log 기록](#-33-ready-상태인-node-log-기록)
- [ 4. Pod Scheduling](#-4-pod-scheduling)
  - [ 4.1. Label 및 selector 개념](#-41-label-및-selector-개념)
  - [ 4.2. 특정 node pod 배포](#-42-특정-node-pod-배포)
  - [ 4.3. 테인트(Taints)와 톨러레이션(Tolerations) 개념](#-43-테인트taints와-톨러레이션tolerations-개념)
- [ 5. ConfigMap \& Secret](#-5-configmap--secret)
  - [ 5.1. kubectl create configmap option 개념](#-51-kubectl-create-configmap-option-개념)
  - [ 5.2. kubectl create secret option 개념](#-52-kubectl-create-secret-option-개념)
  - [ 5.3. openSSL을 이용하여 임의의 TLS, Secret 생성](#-53-openssl을-이용하여-임의의-tls-secret-생성)
  - [ 5.4. configMap 생성](#-54-configmap-생성)
  - [ 5.5. 컨테이너 환경 변수 할당 및 출력](#-55-컨테이너-환경-변수-할당-및-출력)


# <div id='1'/> 1. Deployment
## <div id='1-1'/> 1.1. Deployment, Scale-in/Scale-out 개념
- deployment : k8s에서 애플리케이션 단위를 관리하는 controller
  - kubernetes의 최소 유닛인 pod에 대한 기준 spec을 정의한 object
  - pod의 scale-in/out 되는 기준을 정의
  - pod의 배포되고 update되는 모든 버전을 추적 가능
  - 배포된 pod에 대한 rollback을 수행 가능
  - 즉 개념적으로 deployment=replicaset + pod + history<br> ```replicaset을 만드는 것보다 더 윗단계의 선언(추상표현)```
    - deployment가 replicaset 생성 -> replicaset이 pod 생성
- 스케일 인(scale-in): 레플리카의 수를 줄이는 것
- 스케일 아웃(scale-out): 레플리카의 수를 늘리는 것을 의미
  
## <div id='1-2'/> 1.2. deployment pod 확장
- kubectl 명령을 통해 webserver 라는 이름으로 yaml 을 생성
> kind: deployment <br/>
> Name: webserver <br/>
> 2 replicas <br/>
> label: app_env_stage=dev <br/>
> container name: webserver <br/>
> container image: nginx:1.14 <br/>

```yaml
$ kubectl create deployment webserver --namespace=yang-task --image=nginx:1.14 --replicas=2 --dry-run=client -o yaml > webserver-deployment-yang.yaml

$ vi webserver-deployment-yang.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app_env_stage: dev
  name: webserver
  namespace: yang-task
spec:
  replicas: 2
  selector:
    matchLabels:
      app_env_stage: dev #label 수정 
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app_env_stage: dev #label 수정
    spec:
      containers:
      - image: nginx:1.14
        name: nginx
        resources: {}
status: {}

```

- kubectl 명령을 통해 webserver Deployment의 pod 수를 3개로 확장
```yaml
$ kubectl scale deployment -n yang-task webserver --replicas=3
deployment.apps/webserver scaled

$ kubectl get pod -n yang-task 
NAME                         READY   STATUS    RESTARTS   AGE
webserver-5785c4b975-5vkkd   1/1     Running   0          76s
webserver-5785c4b975-m8tt9   1/1     Running   0          76s
webserver-5785c4b975-zfnfs   1/1     Running   0          16s

```
# <div id='2'/> 2. Rolling Update & Roll back
## <div id='2-1'/> 2.1. Rolling Update / Roll back 개념
- Rolling Update
    - 서비스 중단 없이 애플리케이션을 점진적으로 새로운 버전으로 업데이트하는 방법
    - 새로운 파드를 생성하고 기존 파드를 하나씩 종료하면서 업데이트를 수행(무중단 배포)

- Roll back
    - 문제가 발생했을 경우 이전 버전으로 되돌리는 작업
  
## <div id='2-2'/> 2.2. Rolling Update / Roll back 수행 명령어
- Rolling Update 명령어 예시
```yaml
$ kubectl set image -n yang-task deployment webserver nginx=nginx:1.17  --record
```

- Roll back 명령어
```yaml
$ kubectl rollout undo deployment -n yang-task webserver 
```

## <div id='2-3'/> 2.3. Rolling Update / Roll back 실습
- Deployment를 이용해 nginx 파드를 3개 배포
> name: webserver<br/>
> Image : nginx<br/>
> Image version: 1.16<br/>
> update image version: 1.17<br/>
> label: app=payment, environment=production<br/>
```yaml
$ kubectl create deployment webserver --namespace=yang-task --image=nginx:1.16 --replicas=3 --dry-run=client -o yaml > webserver-yang.yaml

$ vi webserver-yang.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: payment #label 수정
    environment: production #label 수정
  name: webserver
  namespace: yang-task
spec:
  replicas: 3
  selector:
    matchLabels:
      app: payment
      environment: production
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: payment #label 수정
        environment: production #label 수정
    spec:
      containers:
      - image: nginx:1.16
        name: nginx
        resources: {}
status: {}


$ kubectl apply -f webserver-yang.yaml 
deployment.apps/webserver created

$ kubectl describe pod -n yang-task webserver-6855fb78d6-26dlx 
Name:             webserver-6855fb78d6-26dlx
Namespace:        yang-task
Priority:         0
Service Account:  default
Node:             qna-cluster-1/10.100.2.37
Start Time:       Mon, 24 Mar 2025 07:43:51 +0000
Labels:           app=payment
                  environment=production
                  pod-template-hash=6855fb78d6
Annotations:      cni.projectcalico.org/containerID: 2aa6baaad06deec815f28a4f81c09385b6f57a625b43b00e0b0892682a9875fd
                  cni.projectcalico.org/podIP: 10.233.98.138/32
                  cni.projectcalico.org/podIPs: 10.233.98.138/32
Status:           Running
IP:               10.233.98.138
IPs:
  IP:           10.233.98.138
Controlled By:  ReplicaSet/webserver-6855fb78d6
Containers:
  nginx:
    Container ID:   cri-o://643122ffa84e1e1889fb1c354a2c4887141c94a7ad360ebaa4aee8b27f66b294
    Image:          nginx:1.16
---

```


- 컨테이너 이미지 버전을 rolling update
- update record를 기록 
  * --record 옵션을 설정하지 않으면 history를 남기지 않기 때문에 롤백할 수 없다.
```yaml
$ kubectl set image -n yang-task deployment webserver nginx=nginx:1.17  --record
deployment.apps/webserver image updated

# --to-revision=0 <해당 옵션을 사용하면 원하는 시점으로 rollback가능>
$ kubectl rollout status deployment -n yang-task webserver
deployment "webserver" successfully rolled out

$ kubectl describe pod -n yang-task webserver-5f44b7bcfd-l7knn 
Name:             webserver-5f44b7bcfd-l7knn
Namespace:        yang-task
Priority:         0
Service Account:  default
Node:             qna-cluster-4/10.100.2.54
Start Time:       Tue, 25 Mar 2025 02:14:14 +0000
Labels:           app=webserver
                  pod-template-hash=5f44b7bcfd
Annotations:      cni.projectcalico.org/containerID: 783b9c4db38d96dc36601a0ad563385e053420d007857d0af30cf2fff41b4e2e
                  cni.projectcalico.org/podIP: 10.233.68.140/32
                  cni.projectcalico.org/podIPs: 10.233.68.140/32
Status:           Running
IP:               10.233.68.140
IPs:
  IP:           10.233.68.140
Controlled By:  ReplicaSet/webserver-5f44b7bcfd
Containers:
  nginx:
    Container ID:   cri-o://5d21c4a8cb23fe7fac34205df57f6846652bb8cf55afeab9051b4d442e585e72
    Image:          nginx:1.17

---

```



- history 확인 후 컨테이너 이미지를 previous version으로 roll back 
```yaml
$ kubectl rollout history deployment -n yang-task webserver 
REVISION  CHANGE-CAUSE
1         kubectl set image deployment webserver nginx=nginx:1.17 --namespace=yang-task --record=true

$ kubectl rollout undo deployment -n yang-task webserver 
deployment.apps/webserver rolled back

$ kubectl describe pod -n yang-task webserver-6cbfc4858f-sr4nd 
Name:             webserver-6cbfc4858f-sr4nd
Namespace:        yang-task
Priority:         0
Service Account:  default
Node:             qna-cluster-2/10.100.2.160
Start Time:       Tue, 25 Mar 2025 02:16:55 +0000
Labels:           app=webserver
                  pod-template-hash=6cbfc4858f
Annotations:      cni.projectcalico.org/containerID: 1ac96becdc91465c3eb4df6d91803d16b878b11e9e2b557fad0d946137869de3
                  cni.projectcalico.org/podIP: 10.233.85.10/32
                  cni.projectcalico.org/podIPs: 10.233.85.10/32
Status:           Running
IP:               10.233.85.10
IPs:
  IP:           10.233.85.10
Controlled By:  ReplicaSet/webserver-6cbfc4858f
Containers:
  nginx:
    Container ID:   cri-o://de5fb226976872c620862c68ffb5402215a27351428295d237ebbc8c798d4ce3
    Image:          nginx:1.16
---

```


# <div id='3'/> 3. Node management
## <div id='3-1'/> 3.1. kubectl drain option 개념
특정 노드의 스케줄링을 중단하고, 해당 노드에서 실행 중인 모든 파드를 강제로 다른 노드로 이동
  - --ignore-daemonsets : DaemonSet 파드를 제외하고 종료
  - --delete-local-data : 로컬 데이터가 있는 파드도 강제로 종료
  - --delete-emptydir-data : 노드가 비워지면 삭제되는 로컬 데이터인 emptyDir을 사용하는 pod가 있더라도 계속 진행

## <div id='3-2'/> 3.2. 스케줄링 불가능 설정
- 특정 노드를 스케줄링 불가능하게 설정, 해당 노드에서 실행 중인 모든 Pod을 다른 node로 reschedule 
```yaml
$ kubectl drain qna-cluster-4 --ignore-daemonsets --delete-emptydir-data
$ kubectl get node
NAME            STATUS                     ROLES           AGE   VERSION
qna-cluster-1   Ready                      control-plane   34d   v1.30.4
qna-cluster-2   Ready                      control-plane   34d   v1.30.4
qna-cluster-3   Ready                      control-plane   34d   v1.30.4
qna-cluster-4   Ready,SchedulingDisabled   <none>          34d   v1.30.4

```
## <div id='3-3'/> 3.3. Ready 상태인 node log 기록
- Ready 상태(NoSchedule로 taint된 node는 제외)인 node를 찾아 그 수를 notaint_ready_node_yang.log 에 기록
```yaml
$ kubectl get nodes --no-headers | grep -w 'Ready' | grep -v 'SchedulingDisabled' | wc -l > notaint_ready_node_yang.log
$ cat notaint_ready_node_yang.log
3

```
# <div id='4'/> 4. Pod Scheduling

## <div id='4-1'/> 4.1. Label 및 selector 개념
- Label: 리소스에 key-value 형태로 부착하는 데이터
- Selector: 특정 라벨을 기반으로 원하는 리소스를 필터링하는 방법
## <div id='4-2'/> 4.2. 특정 node pod 배포
- Label 및 selector 를 활용하여 다음 조건의 POD을 특정 NODE에 배포 될 수 있도록 구성
> node label: sub-task-node-yang<br/>
> pod name: webserver<br/>
> pod image: nginx<br/>
> node selector: sub-task-node-yang<br/>

```yaml
$ kubectl label node qna-cluster-1 sub-task-node-yang=true

$ kubectl get nodes -L sub-task-node-yang
NAME            STATUS   ROLES           AGE   VERSION   SUB-TASK-NODE-YANG
qna-cluster-1   Ready    control-plane   34d   v1.30.4   true
qna-cluster-2   Ready    control-plane   34d   v1.30.4   
qna-cluster-3   Ready    control-plane   34d   v1.30.4   
qna-cluster-4   Ready    <none>          34d   v1.30.4   

$ kubectl create deployment webserver-sub --namespace=yang-task --image=nginx --dry-run=client -o yaml > node_label_yang.yaml
$ vi node_label_yang.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: node_label_webserver
  name: node_label_webserver
  namespace: yang-task
spec:
  replicas: 1
  selector:
    matchLabels:
      app: node_label_webserver
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: node_label_webserver
    spec:
      nodeSelector:
        sub-task-node-yang: "true"   # 해당 라벨이 있는 노드에만 배포
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}

$ kubectl apply -f node_label_yang.yaml 
deployment.apps/webserver-sub created

$ kubectl get pod -n yang-task -o wide
NAME                             READY   STATUS    RESTARTS   AGE   IP               NODE            NOMINATED NODE   READINESS GATES
webserver-sub-695df78d56-kbhpq   1/1     Running   0          23s   10.233.98.130    qna-cluster-1   <none>           <none>

#node label 삭제 명령어(참고)
$ kubectl label node qna-cluster-1 sub-task-node-yang-
node/qna-cluster-1 unlabeled

$ kubectl get node -L sub-task-node-yang
NAME            STATUS   ROLES           AGE   VERSION   SUB-TASK-NODE-YANG
qna-cluster-1   Ready    control-plane   34d   v1.30.4   
```

## <div id='4-3'/> 4.3. 테인트(Taints)와 톨러레이션(Tolerations) 개념
- Taint
  - Node가 오염되었기 때문에 Pod의 Schedule을 제한
  - Taint는 특정 Node에 대해 특정 Pod만 실행할 수 있도록 역할을 제한하기 위한 목적
  - Taint 예시
```yaml
$ kubectl taint nodes {nodename} {key}={value}:{option}
$ kubectl taint nodes node1 key1=value1:NoSchedule
# taint 해제
$ kubectl taint nodes {nodename} {key}={value}:{option}-
$ kubectl taint nodes node1 key1=value1:NoSchedule-
```
- Toleration: 해당 "오염"을 견딜 수 있는 "내성"
  - Taint 설정이 되어있더라도 해당 Taint에 부합하는 value를 Pod Tolerations에 지정하면 해당 Node에 Schedule 가능
  - Toleration  예시
    - PodSpec에서 파드에 대한 톨러레이션을 지정
    - 위의 kubectl taint 라인에 의해 생성된 테인트와 일치하기 때문에 node1에 스케줄 가능
```yaml
#Toleration 참고 yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations: # tolerations 추가
  - key: "key1"
    operator: "Exists"
    effect: "NoSchedule"
```

# <div id='5'/> 5. ConfigMap & Secret

## <div id='5-1'/> 5.1. kubectl create configmap option 개념
```yaml
$ kubectl create configmap <이름> --from-literal=<키>=<값>
$ kubectl create configmap <이름> --from-file=<경로>
```

## <div id='5-2'/> 5.2. kubectl create secret option 개념
```yaml
$ kubectl create secret generic <이름> --from-literal=<키>=<값>
```

```kubectl create secret tls tls-secret --cert=path/to/cert.crt --key=path/to/cert.key```

## <div id='5-3'/> 5.3. openSSL을 이용하여 임의의 TLS, Secret 생성
- 키와 인증서 생성
```yaml
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=example.com"
..+...+..+..........+...........+....+..+.+............+...........+.+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.......+..+...+............+...+....+...+.................+.+..............+....+..+...+.........................+.........+...+..+.+.................+.......+.....+.......+.....+.........+.+........+..........+.....+......+.......+........+......+.+..+.+.........+..+..........+..............+....+...+........+.+.........+......+.....+......+.............+.....+....+........+....+........+.+......+...+.....+...+...+.......+........+.+........+............+...............+.+...+..+.......+..+...+............................+.........+...+..+...+.......+.........+.........+........+...+....+...+............+......+...........+...+.......+..............+....+...+...+.....+.......+............+..................+...+..+.......+..+.+..+.......+.....+.........+.+...+.....+...+....+.....+.+...............+...+..+.........+.+..+......+.+.....+.+...+..............+...+...................+.........+..+...+.......+.........+......+..+.+......+.....+......+...+......+.+..............+..........+...+.....+...+..........+..+...+...+....+..+.+...............+..+............+.+..+......+......+......+.........+...+.+..+....+...........+...+.+...........+....+...+..+...+.+......+...............+......+..+......+.+...+...+...+.................+.......+..+...+......+....+...+........+......................+............+..+.+.....+...+............+...+....+...+..+...............+...+.......+.....+.+..+............+.+.................+.......+..+.............+..+....+.....+....+..+....+..+....+.....+......+...............+.......+.................+...+.......+...+...+..+......+..........+.....+.........+.+......+...+......+......+........+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
........+.........+...+..+.+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+.+...............+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.......+..........+...+.........+..............+....+...+.....+...+......+.+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----

$ ls
tls.crt  tls.key
```
- TLS Secret 생성
```yaml
$ kubectl create secret tls tls-secret-yang -n yang-task --cert=tls.crt --key=tls.key
secret/tls-secret-yang created

$ kubectl get secret -n yang-task 
NAME              TYPE                DATA   AGE
tls-secret-yang   kubernetes.io/tls   2      17s
```
## <div id='5-4'/> 5.4. configMap 생성 
- 아래 변수 configmap 등록
> DBNAME: mysql<br/>
> USER: admin

```yaml
$ kubectl create configmap app-config-yang --from-literal=DBNAME=mysql --from-literal=USER=admin -n yang-task

$ kubectl get configmaps -n yang-task 
NAME               DATA   AGE
app-config-yang    2      10s
```

## <div id='5-5'/> 5.5. 컨테이너 환경 변수 할당 및 출력
- nginx 컨테이너에 환경변수로 할당
```yaml
$ kubectl create deployment nginx-yang --namespace=yang-task --image=nginx --dry-run=client -o yaml > nginx_yang.yaml


$ vi nginx_yang.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx-yang
  name: nginx-yang
  namespace: yang-task
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-yang
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-yang
    spec:
      containers:
      - image: nginx
        name: nginx
        envFrom: #configmap 추가
          - configMapRef:
              name: app-config-yang
        env:
        - name: tls-secret-yang #secret 추가
          valueFrom:
            secretKeyRef: 
              name: tls-secret-yang
              key: tls.crt
status: {}

$ kubectl apply -f nginx_yang.yaml
deployment.apps/nginx-yang created

$ kubectl get pod -n yang-task 
NAME                          READY   STATUS    RESTARTS   AGE
nginx-yang-6f7d7588cf-9gt5z   1/1     Running   0          16s
```
- 컨테이너에 접근하여 configmap 환경변수와 secret을 출력
```yaml
$ kubectl exec -n yang-task -it nginx-yang-6f7d7588cf-9gt5z -- env | grep DBNAME
DBNAME=mysql

$ kubectl exec -n yang-task -it nginx-yang-6f7d7588cf-9gt5z -- env | grep USER
USER=admin

$ kubectl exec -n yang-task -it nginx-yang-6f7d7588cf-9gt5z -- env | grep tls-secret-yang
tls-secret-yang=-----BEGIN CERTIFICATE-----
```

