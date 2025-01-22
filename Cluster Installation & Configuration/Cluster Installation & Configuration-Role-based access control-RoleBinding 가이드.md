### [kubernetes](https://github.com/ygm0516/kubernetes) > [Cluster Installation & Configuration](https://github.com/ygm0516/kubernetes/tree/main/Cluster%20Installation%20%26%20Configuration) >  Cluster Installation & Configuration-Role-based access control-RoleBinding 가이드


## Cluster Installation & Configuration-Role-based access control-RoleBinding 가이드.md

[Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) <br/>
[Kubectl reference Docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)

## 목차
1. [문서 개요](#1)
    * [1.1. 목적](#1-1)
    * [1.2. 범위](#1-2)
2. [RBAC: ServiceAccount, Role, RoleBinding 설정](#2)
    * [2.1.Namespace 생성](#2-1)
    * [2.2.ServiceAccount 생성](#2-2)
    * [2.3.Role 생성](#2-3)
    * [2.4.RoleBinding 생성](#2-4)
3. [생성 확인 및 검증](#3)
    * [3.1. ServiceAccount 확인](#3-1)
    * [3.2. Role 확인](#3-2)
    * [3.3. RoleBinding 확인](#3-4)
    * [3.4. 권한 테스트](#3-4)


# <div id='1'/> 1. 문서 개요

## <div id='1-1'/> 1.1. 목적
본 문서는 RBAC(Role-based Access Control)를 사용하여 특정 네임스페이스(api-access-yang)에서 Pod들을 모니터링할 수 있는 설정을 수행하는 kubectl 명령어 및 과정에 대하여 기술하였다.

## <div id='1-2'/> 1.1. 범위
모든 명령 수행은 kubectl 을 통해 진행하였다.


# <div id='2'/> 2. RBAC: ServiceAccount, Role, RoleBinding 설정
## <div id='2-1'/>2.1. Namespace 생성
- api-access-yang이라는 네임스페이스를 생성
```
$ kubectl create namespace api-access-yang
```

## <div id='2-2'/>2.2. ServiceAccount 생성
- pod-viewer-yang라는 이름의 ServiceAccount를 생성
```
$ kubectl create serviceaccount pod-viewer-yang -n api-access-yang 
```

## <div id='2-3'/>2.3. Role 생성
- podreader-role-yang 이라는 이름의 Role을 생성하여 watch, list, get 권한을 부여
```
# podreader-role-yang.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: api-access-yang
  name: podreader-role-yang
rules:
- apiGroups: [""] # Core API group
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

```
- 생성 및 생성 확인
```
$ kubectl apply -f podreader-role-yang.yaml
```

## <div id='2-4'/>2.3. RoleBinding 생성
- podreader-rolebinding-yang이라는 RoleBinding을 생성하여 ServiceAccount pod-viewer를 Role podreader-role-yang에 매핑
```
# podreader-rolebinding-yang.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: api-access-yang
  name: podreader-rolebinding-yang
subjects:
- kind: ServiceAccount
  name: pod-viewer-yang
  namespace: api-access-yang
roleRef:
  kind: Role
  name: podreader-role-yang
  apiGroup: rbac.authorization.k8s.io
```
- 생성 및 생성 확인
```
$ kubectl apply -f podreader-rolebinding-yang.yaml
```

# <div id='3'/> 3.생성 확인 및 검증
## <div id='3-1'/>3.1. ServiceAccount 확인
```
$ kubectl get serviceaccount pod-viewer-yang -n api-access-yang
NAME              SECRETS   AGE
pod-viewer-yang   0         10s
```

## <div id='3-2'/>3.2. Role 확인
```
$ kubectl describe role podreader-role-yang -n api-access-yang
Name:         podreader-role-yang
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  pods       []                 []              [get list watch]
```

## <div id='3-3'/>3.3. RoleBinding 확인
```
$ kubectl describe rolebinding podreader-rolebinding-yang -n api-access-yang
Name:         podreader-rolebinding-yang
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  podreader-role-yang
Subjects:
  Kind            Name             Namespace
  ----            ----             ---------
  ServiceAccount  pod-viewer-yang  api-access-yang
```

## <div id='3-4'/>3.4. 권한 테스트
```
$ kubectl auth can-i get pods --as=system:serviceaccount:api-access-yang:pod-viewer-yang -n api-access-yang
yes
$ kubectl auth can-i list pods --as=system:serviceaccount:api-access-yang:pod-viewer-yang -n api-access-yang
yes
$ kubectl auth can-i watch pods --as=system:serviceaccount:api-access-yang:pod-viewer-yang -n api-access-yang
yes
```






