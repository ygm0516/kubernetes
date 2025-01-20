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
    * [4.2. 노드 drain](#4-2)
    * [4.3. kubelet과 kubectl upgrade](#4-3)
    * [4.4. 노드 uncordon](#4-4)

5. [Kubernetes Upgrade 확인](#5)

# <div id='1'/> 1. 문서 개요
## <div id='1-1'/> 1.1. 목적
본 문서는 kubeadm, kubelet, kubectl의 역할과 control plane과 worker node의 Kubernetes 업그레이드 방법에 대하여 기술하였다.
## <div id='1-2'/> 1.2. 범위
Control-plane 과 worker-node-1만 업그레이드 수행하며, 1.29.x버전 업그레이드를 기준으로 작성되었다.

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
$ kubectl get nodes
NAME                  STATUS   ROLES           AGE    VERSION
task-cluster2         Ready    control-plane   146d   v1.28.6
task-cluster2-work1   Ready    <none>          146d   v1.28.6
task-cluster2-work2   Ready    <none>          146d   v1.28.6
task-cluster2-work3   Ready    <none>          146d   v1.28.6
```

## <div id='3-1'/>3.1 kubeadm upgrade
- 업그레이드 버전 결정
	- 목록에서 1.29.x 버전을 찾아서 선택한다.
```
$ apt update
$ apt-cache madison kubeadm
```

- 컨트롤 플레인 노드 업그레이드
```
$ apt-mark unhold kubeadm 
$ apt-get update && apt-get install -y kubeadm=1.29.x-00 
$ apt-mark hold kubeadm
```

- 다운로드할 버전이 정상적으로 다운되었는지 확인
```
$ kubeadm version
$ kubeadm upgrade plan 
```
- "kubeadm upgrade" 호출(Control-plane에서만 진행)
```
$ sudo kubeadm upgrade apply v1.29.x
```
## <div id='3-2'/>3.2. 3.2. 노드 drain

drain : 배출

- 노드에서 유지관리(커널 업그레이드, 하드웨어 유지 관리, kubeadm upgrade 등)을 수행하기 전에 노드에서 모든 pod를 안전하게 제거하는데 사용 가능
- kubernetes의 클러스터 노드 중 하나가 어떠한 이유(리소스 부족, 물리적 결함 등)로 고장이 나면, SchedulingDisabled 또는 Not Ready 상태로 표시되어 더 이상 해당 노드에 Pod가 생성되진 않지만,기존에 있던 Pod는 running로 운영되고 있음

- 해당 노드에 더 이상 파드가 생성되지 않도록 보호하고 문제 해결을 위해 drain을 진행
*drain 명령어는 cordon 이후에 동작함*

```bash
#cordon 적용
$ kubectl cordon [node_name]
$ kubectl drain [node_name]
$ kubectl drain --ignore-daemonsets [node_name]
```
## <div id='3-3'/>3.3. kubelet과 kubectl upgrade

- 모든 컨트롤 플레인 노드에서 kubelet 및 kubectl을 업그레이드

```bash
$ apt-mark unhold kubelet kubectl
$ apt-get update && apt-get install -y kubelet=1.29.x-00 kubectl=1.29.x-00 
$ apt-mark hold kubelet kubectl
```

- kubelet 다시 시작 
```bash
$ sudo systemctl daemon-reload
$ sudo systemctl restart kubelet
```


## <div id='3-4'/>3.4. 노드 uncordon
cordon : 저지선
- 특정 노드를 unSchedule 상태로 만들어서 pod를 스케줄하지 않음
- 기존의 동작하는 pod에 대해서는 간섭하지 않음

*taint - noSchedule과 cordon의 내용이 상당히 비슷
```bash	
#cordon 적용 해제
$ kubectl uncordon [node_name]
```

# <div id='4'/> 4. Worker Node Kubernetes Upgrade
## <div id='4-1'/>4.1 kubeadm upgrade

- 모든 워커 노드 kubeadm 업그레이드
```
$ apt-mark unhold kubeadm
$ apt-get update && apt-get install -y kubeadm=1.29.x-00 
$ apt-mark hold kubeadm
```

- 로컬 kubelet 구성을 업그레이드함
```
$ sudo kubeadm upgrade node
```
## <div id='4-2'/>4.2. 노드 drain

*drain 명령어는 cordon 이후에 동작함*

```bash
#cordon 적용
$ kubectl cordon [node_name]
$ kubectl drain --ignore-daemonsets [node_name]
```
## <div id='4-3'/>4.3. kubelet과 kubectl upgrade

- 모든 컨트롤 플레인 노드에서 kubelet 및 kubectl을 업그레이드

```bash
$ apt-mark unhold kubelet kubectl 
$ apt-get update && apt-get install -y kubelet=1.29.x-00 kubectl=1.29.x-00 
$ apt-mark hold kubelet kubectl
```

- kubelet 다시 시작 
```bash
$ sudo systemctl daemon-reload
$ sudo systemctl restart kubelet
```
## <div id='4-4'/>4.4. 노드 uncordon

```bash	
#cordon 적용 해제
$ kubectl uncordon [node_name]
```

# <div id='5'/> 5. Kubernetes Upgrade 확인
```
kubectl get nodes
```





