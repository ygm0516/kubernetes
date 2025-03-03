### [kubernetes](https://github.com/ygm0516/kubernetes) > [Cluster Installation & Configuration](https://github.com/ygm0516/kubernetes/tree/main/Cluster%20Installation%20%26%20Configuration) > Cluster Installation & Configuration-Kubernetes Upgrade 가이드 

## Cluster Installation & Configuration-Kubernetes Upgrade 가이드.md

[kubeadm 클러스터 업그레이드](https://kubernetes.io/ko/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

## 목차
1. [문서 개요](#1)
    * [1.1. 목적](#1-1)
    * [1.2. 범위](#1-2)
2. [패키지별 용도](#2)
    * [2.1.kubeadm](#2-1)
    * [2.2.kubelet](#2-2)
    * [2.3.kubectl](#2-3)
3. [Control-plane Kubernetes Upgrade](#3)
    * [3.1. kubeadm upgrade](#3-1)
    * [3.2. Control-plane drain](#3-2)
    * [3.3. kubelet과 kubectl upgrade](#3-3)
    * [3.4. Control-plane uncordon](#3-4)
  
4. [Worker Node Kubernetes Upgrade](#4)
    * [4.1. kubeadm upgrade](#4-1)
    * [4.2. Worker node drain](#4-2)
    * [4.3. kubelet과 kubectl upgrade](#4-3)
    * [4.4. Worker node uncordon](#4-4)

5. [Kubernetes Upgrade 확인](#5)

# <div id='1'/> 1. 문서 개요
## <div id='1-1'/> 1.1. 목적
본 문서는 kubeadm, kubelet, kubectl의 역할과 control plane, worker node Kubernetes 업그레이드 방법에 대하여 기술하였다.
## <div id='1-2'/> 1.2. 범위
Control-plane 과 worker-node-1만 업그레이드 수행하며, 1.31.x버전 업그레이드를 기준으로 작성되었다.

# <div id='2'/> 2. 패키지별 용도
## <div id='2-1'/>2.1. kubeadm

<b>kubeadm</b>
<br>
Kubernetes 클러스터를 설치하고 관리하는 데 필요한 기본 도구, 클러스터의 부트스트랩 작업을 자동화하는 역할
클러스터 초기화, 노드 추가 및 업그레이드와 같은 작업을 쉽게 할 수 있도록 명령어를 제공. 
주로 마스터 노드(컨트롤 플레인)를 설정하고 클러스터를 시작하는 데 사용

- 주요 기능
	- 클러스터 초기화 (init)
		- kubeadm init 명령어를 사용하여 컨트롤 플레인을 초기화하고, 클러스터의 기본 구성 요소들을 설치
		- 인증서, 키, 구성 파일 등을 생성하고, 필수적인 컨트롤러 컴포넌트들을 설정
	- 노드 추가(join)
		- kubeadm join 명령어를 사용하여 워커 노드를 기존 클러스터에 추가
		- 이를 통해 워커 노드가 마스터 노드에 연결되어 작업을 처리 가능
	- 클러스터 업그레이드
		- Kubernetes의 새로운 버전이 릴리스될 때, kubeadm upgrade 명령어를 사용하여 클러스터를 업그레이드 가능

## <div id='2-2'/>2.2. kubelet

<b>kubelet</b>
<br>
Kubernetes 클러스터에서 각 노드에 상주하며 Pod와 컨테이너를 관리하는 에이전트
마스터 노드의 API 서버에서 내려오는 명령을 받아서 해당 노드에서 컨테이너가 올바르게 실행되는지 지속적으로 모니터링하고 관리하는 역할
Kubernetes의 기본 구성 요소 중 하나로, Pod의 상태를 보고하고 필요한 경우 컨테이너를 다시 시작하는 등 작업을 수행

- 주요 기능
	- Pod 및 컨테이너 관리
		- Pod의 정의를 받아 해당 Pod를 생성하고, 올바르게 실행되고 있는지 확인
		- 컨테이너의 상태를 주기적으로 확인하며, 상태가 이상할 경우 컨테이너를 재시작하여 상태를 유지하도록 
	- 노드 상태 보고
		- 마스터 노드의 API 서버에 노드의 상태와 자원 사용량(CPU, 메모리 등)을 주기적으로 보고
		- 클러스터 관리자가 노드의 상태를 모니터링
	- 로깅 및 진단
		- 로그 파일을 관리하고, 시스템의 진단 정보를 수집하여 클러스터의 상태를 확인하는 데 도움

## <div id='2-3'/>2.3. kubectl

<b>kubectl</b>
<br>
Kubernetes 클러스터와 상호작용하기 위한 명령줄 도구
클러스터 내 리소스 관리, 배포, 디버깅 등을 수행할 수 있는 가장 중요한 도구 중 하나
Kubernetes API 서버와 통신하여 클러스터 상태를 조회하거나 관리할 수 있으며, Kubernetes 리소스를 생성, 조회, 수정, 삭제하는 다양한 작업을 수행 가능

- 주요 기능
	- Kubernetes 리소스 관리
		- 클러스터 내에서 Pod, Service, Deployment, ConfigMap 등과 같은 리소스를 생성, 조회, 업데이트, 삭제 가능
		- 컨테이너의 상태를 주기적으로 확인하며, 상태가 이상할 경우 컨테이너를 재시작하여 상태를 유지하도록 
	- 클러스터 상태 모니터링
		- 노드, 네임스페이스, 컨테이너 등의 상태를 확인하고, 로그를 조회하여 애플리케이션 상태를 진단
	- 배포 관리
		- 애플리케이션 배포, 확장, 롤백 등의 작업을 쉽게 수행

# <div id='3'/> 3. Control-plane Kubernetes Upgrade
- 추상적인 업그레이드 작업 절차
	1) 기본 컨트롤 플레인 노드를 업그레이드한다.
	2) 추가 컨트롤 플레인 노드를 업그레이드한다.
	3) 워커(worker) 노드를 업그레이드한다.

- 기존 버전 확인 
```bash
$ kubectl get node
NAME                STATUS                     ROLES           AGE    VERSION
ta-task-cluster-1   Ready		       control-plane   102m   v1.30.4
ta-task-cluster-2   Ready                      <none>          101m   v1.30.4
ta-task-cluster-3   Ready                      <none>          101m   v1.30.4
ta-task-cluster-4   Ready                      <none>          101m   v1.30.4

```

## <div id='3-1'/>3.1 kubeadm upgrade
- 업그레이드 버전 결정
	- 목록에서 1.31.x 버전을 찾아서 선택한다.

>  ```sudo apt-cache madison kubeadm``` 명령어 입력시 ```N: Unable to locate package kubeadm```에러 발생 해결 방법
```
$ sudo apt-get install -y apt-transport-https ca-certificates curl gpg
$ sudo mkdir -p -m 755 /etc/apt/keyrings
$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
$ sudo apt update
$ sudo apt-cache madison kubeadm
```

- 컨트롤 플레인 노드 업그레이드
```
$ sudo apt-mark unhold kubeadm 
$ sudo apt-get update && apt-get install -y kubeadm='1.31.5-1.1'
$ sudo apt-mark hold kubeadm
```

- 다운로드할 버전이 정상적으로 다운되었는지 확인
```
$ dpkg -L kubeadm
$ dpkg-query -L kubeadm
$ /usr/bin/kubeadm version
$ echo $PATH
$ /usr/local/bin/kubeadm version
$ sudo cp /usr/local/bin/kubeadm ~/kubeadm
$ sudo mv /usr/bin/kubeadm /usr/local/bin/kubeadm
$ kubeadm version
$ kubeadm upgrade plan 
```
- "kubeadm upgrade" upgrade 적용 
```
$ sudo kubeadm upgrade apply v1.31.5
```
## <div id='3-2'/>3.2. 3.2. Control-plane drain
업그레이드 전 node를 drain을 해준다.
drain 상태에서는 Pod가 더는 할당되지 않게 taint 시킬 뿐 아니라 노드 내 존재하는 Pod들을 Evict(퇴거)시킨다.
해당 노드에 더 이상 파드가 생성되지 않도록 보호하고 문제 해결을 위해 drain을 진행한다.


```bash
#cordon 적용
$ kubectl drain --ignore-daemonsets [Control_plane_name]
```
## <div id='3-3'/>3.3. kubelet과 kubectl upgrade
```kubectl get nodes``` 명령어를 통해 보았을 때 아직 버전 업그레이드가 진행되지 않은 것처럼 보인다.
```
여기에 명령어 결과 삽입
```
이는 kubelet이 아직 업그레이드가 되지 않아서 그렇다.

- 모든 컨트롤 플레인 노드에서 kubelet 및 kubectl을 업그레이드를 진행한다.
```bash
$ sudo apt-mark unhold kubelet kubectl
$ sudo apt-get update && sudo apt-get install -y kubelet='1.31.5-1.1' kubectl='1.31.5-1.1'
$ sudo apt-mark hold kubelet kubectl
```

- kubelet 재시작
```bash
$ sudo systemctl daemon-reload
$ sudo systemctl restart kubelet
```


## <div id='3-4'/>3.4. Control-plane uncordon
- uncordon 명령을 통해 pod가 다시 스케줄링 될 수 있게 설정한다.
```bash	
#cordon 적용 해제
$ kubectl uncordon [Control_plane_name]
```

# <div id='4'/> 4. Worker Node Kubernetes Upgrade
## <div id='4-1'/>4.1 kubeadm upgrade


- Worker Node kubeadm 업그레이드(이 작업은 업그레이드할 워커 노드에서 진행한다)
```
$ sudo apt-mark unhold kubeadm
$ sudo apt-get update && sudo apt-get install -y kubeadm='1.31.5-1.1'
$ sudo apt-mark hold kubeadm
```

- kubelet configuration upgrade
```
$ sudo kubeadm upgrade node
```

## <div id='4-2'/>4.2. Worker Node drain
- kubectl 명령이기에 Worker node가 아닌 Master Node에서 Drain 작업을 진행해야 한다.
```bash
$ kubectl drain --ignore-daemonsets ta-task-cluster-2
```
## <div id='4-3'/>4.3. kubelet과 kubectl upgrade

>  ```sudo apt-mark unhold kubelet kubectl``` 명령어 입력시 ```N: Unable to locate package kubectl...```에러 발생 해결 방법
```
$ sudo apt-get install -y apt-transport-https ca-certificates curl gpg
$ sudo mkdir -p -m 755 /etc/apt/keyrings
$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
$ sudo apt update
```

- kubelet 및 kubect upgrade
```bash

$ sudo apt-mark unhold kubelet kubectl 
$ sudo apt-get update && sudo apt-get install -y kubelet='1.31.5-1.1' kubectl='1.31.5-1.1'
$ sudo apt-mark hold kubelet kubectl
```

- kubelet 재시작
```bash
$ sudo systemctl daemon-reload
$ sudo systemctl restart kubelet
```
## <div id='4-4'/>4.4. Worker Node uncordon
```bash	
#cordon 적용 해제
$ kubectl uncordon [Worker_node_name]
```

# <div id='5'/> 5. Kubernetes Upgrade 확인
```
$ kubectl get node
NAME                STATUS                     ROLES           AGE    VERSION
ta-task-cluster-1   Ready                      control-plane   120m   v1.31.5
ta-task-cluster-2   Ready                      <none>          119m   v1.31.5
ta-task-cluster-3   Ready                      <none>          119m   v1.30.4
ta-task-cluster-4   Ready                      <none>          119m   v1.30.4

```





