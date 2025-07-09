# Troubleshooting 

- [Troubleshooting](#troubleshooting)
  - [1. 모니터링](#1-모니터링)
    - [1-1. kube-api 의 로그 모니터링 후 'error' 오류가 있는 로그 라인 추출(Extract)해서 /subtask/CUSTOM-LOG001 파일에 저장후 출력](#1-1-kube-api-의-로그-모니터링-후-error-오류가-있는-로그-라인-추출extract해서-subtaskcustom-log001-파일에-저장후-출력)
    - [1-2. kubectl 명령을 사용하여 POD중 CPU를 가장 많이 사용하는 순서대로 출력](#1-2-kubectl-명령을-사용하여-pod중-cpu를-가장-많이-사용하는-순서대로-출력)
    - [1-3. kubectl 명령을 사용하여 NODE중 메모리를 가장 적게 사용하는 순서대로 출력](#1-3-kubectl-명령을-사용하여-node중-메모리를-가장-적게-사용하는-순서대로-출력)
    - [1-4. kubectl 명령을 사용하여 클러스터에 구성된 모든 PV를 capacity별로 sort하여 출력 (json 포멧 활용)](#1-4-kubectl-명령을-사용하여-클러스터에-구성된-모든-pv를-capacity별로-sort하여-출력-json-포멧-활용)
    - [1-5. kubectl 명령을 사용하여 클러스터 내 POD의 전체 레이블을 확인 후 배포된 POD중 레이블을 임의로 선택하고, 선택한 레이블을 사용하는 Pod들 중 CPU 소비율이 가장 높은 Pod의 이름을 찾아서 출력](#1-5-kubectl-명령을-사용하여-클러스터-내-pod의-전체-레이블을-확인-후-배포된-pod중-레이블을-임의로-선택하고-선택한-레이블을-사용하는-pod들-중-cpu-소비율이-가장-높은-pod의-이름을-찾아서-출력)


## 1. 모니터링
### 1-1. kube-api 의 로그 모니터링 후 'error' 오류가 있는 로그 라인 추출(Extract)해서 /subtask/CUSTOM-LOG001 파일에 저장후 출력
```bash
$ mkdir subtask
$ kubectl logs -n kube-system -l component=kube-apiserver | grep Error | tee subtask/CUSTOM-LOG001
$ cat subtask/CUSTOM-LOG001
E0707 00:23:50.932402       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
E0707 00:25:07.992222       1 conn.go:339] Error on socket receive: read tcp 172.16.1.12:6443->172.16.100.17:34130: use of closed network connection
E0707 00:26:09.204026       1 conn.go:339] Error on socket receive: read tcp 172.16.1.12:6443->172.16.100.17:3646: use of closed network connection
E0707 00:26:21.781054       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
E0707 00:33:26.992197       1 conn.go:339] Error on socket receive: read tcp 172.16.1.12:6443->172.16.100.16:42056: use of closed network connection
E0707 04:17:57.349331       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
E0707 04:26:06.914941       1 conn.go:339] Error on socket receive: read tcp 172.16.1.12:6443->172.16.100.16:35804: use of closed network connection
E0707 00:19:58.527287       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
E0707 00:22:32.628081       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
E0707 00:25:11.107783       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
E0707 00:26:27.700143       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
E0707 00:27:01.268119       1 conn.go:339] Error on socket receive: read tcp 172.16.1.13:6443->172.16.100.16:3152: use of closed network connection
E0707 04:23:01.411887       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
E0707 04:27:04.956966       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
E0707 04:28:35.855570       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
E0707 04:29:19.042021       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
E0707 04:30:50.684625       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"

```
### 1-2. kubectl 명령을 사용하여 POD중 CPU를 가장 많이 사용하는 순서대로 출력
```bash
$ kubectl top pod --all-namespaces --sort-by=cpu
NAMESPACE        NAME                                              CPU(cores)   MEMORY(bytes)   
kube-system      kube-apiserver-qna-cluster-003                    29m          490Mi           
kube-system      kube-apiserver-qna-cluster-001                    19m          427Mi           
kube-system      calico-node-5tg49                                 17m          158Mi           
kube-system      kube-apiserver-qna-cluster-002                    17m          363Mi           
kube-system      calico-node-nxb5n                                 13m          157Mi           
kube-system      calico-node-5g9hz                                 13m          157Mi           
kube-system      calico-node-m5h66                                 11m          157Mi           
kube-system      kube-controller-manager-qna-cluster-001           7m           89Mi            
kube-system      kube-proxy-2h4tl                                  6m           42Mi            
kube-system      kube-proxy-h8lv7                                  5m           41Mi            
kube-system      kube-proxy-5s2fj                                  4m           23Mi            
kube-system      kube-proxy-m9wx9                                  4m           45Mi            
kube-system      metrics-server-6c8bff4c-cf6cx                     3m           43Mi            
kube-system      nodelocaldns-9mp4d                                2m           22Mi            
kube-system      kube-scheduler-qna-cluster-001                    2m           41Mi            
sy-subtask       nginx                                             2m           7Mi             
metallb-system   speaker-w86df                                     2m           36Mi            
metallb-system   speaker-8xtxw                                     2m           37Mi            
kube-system      kube-scheduler-qna-cluster-003                    2m           43Mi            
kube-system      kube-scheduler-qna-cluster-002                    2m           46Mi            
default          nfs-subdir-external-provisioner-8989f7c84-pds7b   1m           7Mi             
kube-system      nodelocaldns-xkfqm                                1m           26Mi            
kube-system      coredns-d665d669-6hqxx                            1m           34Mi            
ingress-nginx    ingress-nginx-controller-6887c6d764-95r4d         1m           52Mi            
kube-system      kube-controller-manager-qna-cluster-002           1m           43Mi            
kube-system      nodelocaldns-9sjv4                                1m           26Mi            
kube-system      nodelocaldns-wn5ck                                1m           13Mi            
kube-system      calico-kube-controllers-5db5978889-ns8z7          1m           52Mi            
linkerd-smi      smi-adaptor-78f8cfc865-7br66                      1m           12Mi            
metallb-system   controller-74b6dc8f85-n5bg2                       1m           17Mi            
kube-system      coredns-d665d669-dzpzt                            1m           32Mi            
metallb-system   speaker-jzkks                                     1m           36Mi            
metallb-system   speaker-t5nl6                                     1m           18Mi            
kube-system      dns-autoscaler-5cb4578f5f-rpsvc                   1m           21Mi            
kube-system      kube-controller-manager-qna-cluster-003           1m           40Mi         
```
### 1-3. kubectl 명령을 사용하여 NODE중 메모리를 가장 적게 사용하는 순서대로 출력
```bash
$ kubectl top node --sort-by=memory | sort -k5 -n
NAME              CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
qna-cluster-004   44m          3%     1450Mi          20%       
qna-cluster-003   68m          4%     1928Mi          63%       
qna-cluster-002   82m          5%     1951Mi          64%       
qna-cluster-001   83m          5%     2226Mi          73%   
```

### 1-4. kubectl 명령을 사용하여 클러스터에 구성된 모든 PV를 capacity별로 sort하여 출력 (json 포멧 활용)
- jq 도구 필요. 없다면 설치 필요 <br>
  `sudo apt install jq`
```bash
$ kubectl get pv -o json | jq '.items | sort_by(.spec.capacity.storage) | .[] | {name: .metadata.name, capacity: .spec.capacity.storage}'
{
  "name": "pvc-e6d36418-497e-492f-9fde-67328981f16a",
  "capacity": "10Mi"
}
{
  "name": "pv001-yang",
  "capacity": "1Gi"
}

```

### 1-5. kubectl 명령을 사용하여 클러스터 내 POD의 전체 레이블을 확인 후 배포된 POD중 레이블을 임의로 선택하고, 선택한 레이블을 사용하는 Pod들 중 CPU 소비율이 가장 높은 Pod의 이름을 찾아서 출력

```bash
# 전체 Pod 레이블 확인
$ kubectl get pods --all-namespaces --show-labels
NAMESPACE        NAME                                              READY   STATUS      RESTARTS   AGE    LABELS
default          nfs-subdir-external-provisioner-8989f7c84-pds7b   1/1     Running     0          16d    app=nfs-subdir-external-provisioner,pod-template-hash=8989f7c84,release=nfs-subdir-external-provisioner
ingress-nginx    ingress-nginx-controller-6887c6d764-95r4d         1/1     Running     0          16d    app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx,app.kubernetes.io/part-of=ingress-nginx,app.kubernetes.io/version=1.12.0,pod-template-hash=6887c6d764
kube-system      calico-kube-controllers-5db5978889-ns8z7          1/1     Running     7          18d    k8s-app=calico-kube-controllers,pod-template-hash=5db5978889
kube-system      calico-node-5g9hz                                 1/1     Running     4          19d    controller-revision-hash=65b4898996,k8s-app=calico-node,pod-template-generation=1
kube-system      calico-node-5tg49                                 1/1     Running     8          19d    controller-revision-hash=65b4898996,k8s-app=calico-node,pod-template-generation=1
kube-system      calico-node-m5h66                                 1/1     Running     8          19d    controller-revision-hash=65b4898996,k8s-app=calico-node,pod-template-generation=1
kube-system      calico-node-nxb5n                                 1/1     Running     8          19d    controller-revision-hash=65b4898996,k8s-app=calico-node,pod-template-generation=1
kube-system      coredns-d665d669-6hqxx                            1/1     Running     8          19d    k8s-app=kube-dns,pod-template-hash=d665d669
kube-system      coredns-d665d669-dzpzt                            1/1     Running     6          18d    k8s-app=kube-dns,pod-template-hash=d665d669
kube-system      dns-autoscaler-5cb4578f5f-rpsvc                   1/1     Running     8          19d    k8s-app=dns-autoscaler,pod-template-hash=5cb4578f5f
kube-system      kube-apiserver-qna-cluster-001                    1/1     Running     8          19d    component=kube-apiserver,tier=control-plane
kube-system      kube-apiserver-qna-cluster-002                    1/1     Running     8          19d    component=kube-apiserver,tier=control-plane
kube-system      kube-apiserver-qna-cluster-003                    1/1     Running     8          19d    component=kube-apiserver,tier=control-plane
kube-system      kube-controller-manager-qna-cluster-001           1/1     Running     11         19d    component=kube-controller-manager,tier=control-plane
kube-system      kube-controller-manager-qna-cluster-002           1/1     Running     12         19d    component=kube-controller-manager,tier=control-plane
kube-system      kube-controller-manager-qna-cluster-003           1/1     Running     11         19d    component=kube-controller-manager,tier=control-plane
kube-system      kube-proxy-2h4tl                                  1/1     Running     8          19d    controller-revision-hash=d78758f9,k8s-app=kube-proxy,pod-template-generation=1
kube-system      kube-proxy-5s2fj                                  1/1     Running     4          19d    controller-revision-hash=d78758f9,k8s-app=kube-proxy,pod-template-generation=1
kube-system      kube-proxy-h8lv7                                  1/1     Running     8          19d    controller-revision-hash=d78758f9,k8s-app=kube-proxy,pod-template-generation=1
kube-system      kube-proxy-m9wx9                                  1/1     Running     8          19d    controller-revision-hash=d78758f9,k8s-app=kube-proxy,pod-template-generation=1
kube-system      kube-scheduler-qna-cluster-001                    1/1     Running     10         19d    component=kube-scheduler,tier=control-plane
kube-system      kube-scheduler-qna-cluster-002                    1/1     Running     10         19d    component=kube-scheduler,tier=control-plane
kube-system      kube-scheduler-qna-cluster-003                    1/1     Running     11         19d    component=kube-scheduler,tier=control-plane
kube-system      metrics-server-6c8bff4c-cf6cx                     1/1     Running     6          18d    app.kubernetes.io/name=metrics-server,pod-template-hash=6c8bff4c,version=v0.7.0
kube-system      nodelocaldns-9mp4d                                1/1     Running     10         19d    controller-revision-hash=76c95d7d99,k8s-app=node-local-dns,pod-template-generation=1
kube-system      nodelocaldns-9sjv4                                1/1     Running     9          19d    controller-revision-hash=76c95d7d99,k8s-app=node-local-dns,pod-template-generation=1
kube-system      nodelocaldns-wn5ck                                1/1     Running     4          19d    controller-revision-hash=76c95d7d99,k8s-app=node-local-dns,pod-template-generation=1
kube-system      nodelocaldns-xkfqm                                1/1     Running     10         19d    controller-revision-hash=76c95d7d99,k8s-app=node-local-dns,pod-template-generation=1
linkerd-smi      smi-adaptor-78f8cfc865-7br66                      1/1     Running     0          16d    component=smi-adaptor,linkerd.io/extension=smi,pod-template-hash=78f8cfc865
metallb-system   controller-74b6dc8f85-n5bg2                       1/1     Running     0          16d    app=metallb,component=controller,pod-template-hash=74b6dc8f85
metallb-system   speaker-8xtxw                                     1/1     Running     8          19d    app=metallb,component=speaker,controller-revision-hash=87d554745,pod-template-generation=1
metallb-system   speaker-jzkks                                     1/1     Running     8          19d    app=metallb,component=speaker,controller-revision-hash=87d554745,pod-template-generation=1
metallb-system   speaker-t5nl6                                     1/1     Running     4          19d    app=metallb,component=speaker,controller-revision-hash=87d554745,pod-template-generation=1
metallb-system   speaker-w86df                                     1/1     Running     8          19d    app=metallb,component=speaker,controller-revision-hash=87d554745,pod-template-generation=1
sy-subtask       nginx                                             2/2     Running     0          9d     run=nginx
testns           test-denied                                       0/1     Completed   0          3d3h   run=test-denied



$ kubectl top pod --all-namespaces --selector=app=metallb --sort-by=cpu
NAMESPACE        NAME                          CPU(cores)   MEMORY(bytes)   
metallb-system   speaker-8xtxw                 2m           37Mi            
metallb-system   speaker-jzkks                 2m           35Mi            
metallb-system   speaker-t5nl6                 2m           18Mi            
metallb-system   speaker-w86df                 2m           35Mi            
metallb-system   controller-74b6dc8f85-n5bg2   1m           17Mi  

# 가장 높은 Pod 이름 추출
$ kubectl top pod --all-namespaces --selector=app=metallb --sort-by=cpu | awk 'NR==2 {print $2}'
speaker-t5nl6

```