## Cluster Installation & Configuration-ETCD 백업 & 복구 가이드.md


[Kubernetes 운영을 위한 etcd 기본 동작 원리의 이해](https://tech.kakao.com/posts/484) <br/>
[Backing up an etcd cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)<br/>
[볼륨 스냅샷](https://kubernetes.io/ko/docs/concepts/storage/volume-snapshots/)<br/>

## 목차
1. [문서 개요](#1)
    * [1.1. 목적](#1-1)
    * [1.2. 범위](#1-2)
2. [etcd](#2)
    * [2.1. etcd란.](#2-1)
    * [2.2. kubernetes 운영을 위한 etcd 기본 동작 원리의 이해](#2-1)
3. [ETCD 백업](#3)
    * [3.1. etcdctl 세팅](#3-1)
    * [3.2. ETCD 백업](#3-2)
4. [ETCD 복구](#4)
    * [4.1. snapshot 복구](#4-1)
	* [4.2. 복구 확인](#4-2)

# <div id='1'/> 1. 문서 개요

## <div id='1-1'/> 1.1. 목적

## <div id='1-2'/> 1.1. 범위
본 가이드는 단일 ETCD를 기준으로 백업 및 복구 수행 방법에 대해 기술하였다.



# <div id='2'/> 2. etcd

## <div id='2-1'/>2.1. etcd란.

#### etcd
`key:value` 형태의 데이터를 저장하는 스토리지(분산형)
분산시스템에서 중요한 데이터를 저장할 때 사용할 수 있는, 믿을 수 있는 key value 분산 저장소
분산 컴퓨팅 환경에서 서버가 몇개 다운되어도 잘동작하는 시스템을 만들고자할때 선택하는 방법

- 저장 데이터
	- nodes
	- pods
	- configs
	- secrets
	- accounts
	- roles
	- bindings

- 쿠버네티스에서 etcd의 역할
etcd 데이터 저장소는 노드, 파드, 구성, 암호, 계정, 역할, 바인딩 같은 클러스터 관련 정보를 저장함
(node,pod,config,secret,account,role,binding...)
`kubectl get pod`와 같은 `command`들은 모두 etcd로부터 받는 정보

etcd가 다운될 경우 클러스터는 제대로 동작하지 못함

추가적으로, cluster에 node를 추가하거나, pod 배포, replica set의 업데이트 등과 같은 부분들에 대한 정보도 etcd에 업데이트


## <div id='2-2'/>2.2. kubernetes 운영을 위한 etcd 기본 동작 원리의 이해

etcd는 Replicated state machine라는 방법을 통해 분산된 환경 구성을 제공함

- RSM(Replicated state machine)
RSM : command가 들어있는 log 단위로 데이터를 처리함
모든 노드가 독립적으로 구성된것이 아닌 RSM이라는 형태로 묶여서 동작하기 때문에 RSM에는 어떤 상태가 위의 전제조건들을 만족하는 올바른 상태인지를 합의하기 위한 모종의 방법이 필요

이렇게 분산 환경에서 상태를 공유하는 알고리즘을 Consensus Algorithm이라고 부르며, etcd는 그 중 Raft Algorithm을 사용



# <div id='3'/> 3.ETCD 백업
## <div id='3-1'/>3.1. etcdctl 세팅

etcdctl 명령어 사용을 위해서는 etcd의 버전, 엔드포인트, 인증 정보를 함께 제공하여야함
구성한 클러스터의 파일에서 변수 확인

※ endpoint, cacert, cert, key 정보 확인
```bash
$ cd /etc/kubernetes/manifests
$ sudo cat kube-apiserver.yaml
$ ps -ef | grep kube | grep etcd
```

## <div id='3-2'/>3.2. ETCD 백업

- /data/etcd-snapshot-yang.db라는 이름으로 ETCD 스냅샷 생성
```bash
$ sudo su
$ etcdctl --endpoints=https://127.0.0.1:2379 --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> snapshot save /data/etcd-snapshot-yang.db
{"level":"info","ts":"2025-01-20T04:41:29.643054Z","caller":"snapshot/v3_snapshot.go:65","msg":"created temporary db file","path":"/data/etcd-snapshot-yang.db.part"}
{"level":"info","ts":"2025-01-20T04:41:29.67065Z","logger":"client","caller":"v3@v3.5.10/maintenance.go:212","msg":"opened snapshot stream; downloading"}
{"level":"info","ts":"2025-01-20T04:41:29.67073Z","caller":"snapshot/v3_snapshot.go:73","msg":"fetching snapshot","endpoint":"https://127.0.0.1:2379"}
{"level":"info","ts":"2025-01-20T04:41:30.546221Z","logger":"client","caller":"v3@v3.5.10/maintenance.go:220","msg":"completed snapshot read; closing"}
{"level":"info","ts":"2025-01-20T04:41:31.023806Z","caller":"snapshot/v3_snapshot.go:88","msg":"fetched snapshot","endpoint":"https://127.0.0.1:2379","size":"29 MB","took":"1 second ago"}
{"level":"info","ts":"2025-01-20T04:41:31.024024Z","caller":"snapshot/v3_snapshot.go:97","msg":"saved","path":"/data/etcd-snapshot-yang.db"}
Snapshot saved at /data/etcd-snapshot-yang.db

```

- 스냅샷 생성 확인
```
$ cd /data/
$ ls 
etcd-snapshot-yang.db
```

## <div id='3-3'/>3.3. nginx application 배포
- etcd-yang Namespace 생성
```
kubectl create ns etcd-yang
```

- nginx application 배포
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: etcd-yang
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: etcd-yang
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
    - port: 80
```


# <div id='4'/> 4. ETCD 복구
## <div id='4-1'/>4.1. snapshot 복구
- /data/etcd-snapshot-yang.db 생성한 etcd 스냅샷을 사용하여 복구
- 스냅샷을 떴던 시점으로 다시 되돌리겠다는 것을 의미함
```bash
$ sudo etcdctl --data-dir /data/etcd-new-yang snapshot restore /data/etcd-snapshot-yang.db
Deprecated: Use `etcdutl snapshot restore` instead.

2025-01-20T04:42:59Z	info	snapshot/v3_snapshot.go:260	restoring snapshot	{"path": "/data/etcd-snapshot-yang.db", "wal-dir": "/data/etcd-new-yang/member/wal", "data-dir": "/data/etcd-new-yang", "snap-dir": "/data/etcd-new-yang/member/snap"}
2025-01-20T04:42:59Z	info	membership/store.go:141	Trimming membership information from the backend...
2025-01-20T04:43:00Z	info	membership/cluster.go:421	added member	{"cluster-id": "cdf818194e3a8c32", "local-member-id": "0", "added-peer-id": "8e9e05c52164694d", "added-peer-peer-urls": ["http://localhost:2380"]}
2025-01-20T04:43:00Z	info	snapshot/v3_snapshot.go:287	restored snapshot	{"path": "/data/etcd-snapshot-yang.db", "wal-dir": "/data/etcd-new-yang/member/wal", "data-dir": "/data/etcd-new-yang", "snap-dir": "/data/etcd-new-yang/member/snap"}
```

- tree 명령 사용하여 생성 확인
```
$ sudo tree /data/etcd-new-yang/
/data/etcd-new-yang/
└── member
    ├── snap
    │   ├── 0000000000000001-0000000000000001.snap
    │   └── db
    └── wal
        └── 0000000000000000-0000000000000000.wal

3 directories, 3 files
```

- etcd 디렉터리 교체
```
$ sudo systemctl stop etcd
$ sudo mv /var/lib/etcd/ /var/lib/etcd.bak
$ sudo mv /data/etcd-new-yang/ /var/lib/etcd
$ sudo chown -R etcd:etcd /var/lib/etcd
```

- etcd 서비스 재시작
```
$ sudo systemctl start etcd
$ sudo systemctl status etcd
```


## <div id='4-2'/>4.2. 복구 확인

- nginx 배포 전으로 돌아가 있음
```
$ kubectl get pod -n etcd-yang
No resources found in etcd-yang namespace.
```










