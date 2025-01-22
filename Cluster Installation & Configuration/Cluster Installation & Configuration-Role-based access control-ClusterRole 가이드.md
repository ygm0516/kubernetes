### [kubernetes](https://github.com/ygm0516/kubernetes) > [Cluster Installation & Configuration](https://github.com/ygm0516/kubernetes/tree/main/Cluster%20Installation%20%26%20Configuration) >  Cluster Installation & Configuration-Role-based access control-ClusterRole 가이드


## Cluster Installation & Configuration-Role-based access control-ClusterRole 가이드.md

[Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) <br/>
[Kubectl reference Docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)

## 목차
1. [문서 개요](#1)
    * [1.1. 목적](#1-1)
    * [1.2. 범위](#1-2)
2. [RBAC: ServiceAccount, ClusterRole, ClusterRoleBinding 설정](#2)
    * [2.1.Namespace 생성](#2-1)
    * [2.2.ServiceAccount 생성](#2-2)
    * [2.3.ClusterRole  생성](#2-3)
    * [2.4.ClusterRoleBinding 생성](#2-4)
3. [생성 확인 및 검증](#3)
    * [3.1. ServiceAccount 확인](#3-1)
    * [3.2. Role 확인](#3-2)
    * [3.3. RoleBinding 확인](#3-4)
    * [3.4. 권한 테스트](#3-4)


# <div id='1'/> 1. 문서 개요

## <div id='1-1'/> 1.1. 목적
본 문서는 작업 Context에서 애플리케이션 배포를 위해 새로운 ClusterRole을 생성하고 특정 Namespace의 ServiceAccount에 바인드 작업을 수행하는 kubectl 명령어 및 과정에 대하여 기술하였다.

## <div id='1-2'/> 1.1. 범위
모든 명령 수행은 kubectl 을 통해 진행하였다.

# <div id='2'/> 2. RBAC: ServiceAccount, ClusterRole, ClusterRoleBinding 설정

## <div id='2-1'/>2.1. Namespace 생성
- api-access-yang이라는 네임스페이스를 생성
```
$ kubectl create namespace api-access-yang
```

## <div id='2-2'/>2.2. ServiceAccount 생성
- api-access-yang 네임스페이스에 cicd-token-yang 이라는 이름의 ServiceAccount를 생성
```
kubectl create serviceaccount cicd-token-yang -n api-access-yang
```

- 생성 확인
```
kubectl get serviceaccount cicd-token-yang -n api-access-yang
```
## <div id='2-3'/>2.3. ClusterRole 생성
- deployment-clusterrole-yang이라는 이름의 ClusterRole을 생성하여 리소스 Deployment, StatefulSet, DaemonSet에 대해 Create 권한만 부여
```
# deployment-clusterrole-yang.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: deployment-clusterrole-yang
rules:
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets", "daemonsets"]
  verbs: ["create"]
```

- 생성
```
kubectl apply -f deployment-clusterrole-yang.yaml
```

## <div id='2-4'/>2.3. ClusterRoleBinding 생성

- deployment-clusterrolebinding-yang을 api-access-yang 네임스페이스의 cicd-token-yang ServiceAccount에 바인딩합니다.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: deployment-clusterrolebinding-yang
subjects:
- kind: ServiceAccount
  name: cicd-token-yang
  namespace: api-access-yang
roleRef:
  kind: ClusterRole
  name: deployment-clusterrole-yang
  apiGroup: rbac.authorization.k8s.io
```

- 생성 및 생성 확인
```
kubectl apply -f deployment-clusterrolebinding-yang.yaml
```

# <div id='3'/> 3.생성 확인 및 검증
## <div id='3-1'/>3.1. ServiceAccount 확인
```
$ kubectl get serviceaccount cicd-token-yang -n api-access-yang

NAME              SECRETS   AGE
cicd-token-yang   0         3m43s
```
## <div id='3-2'/>3.2. ClusterRole 확인
```
$ kubectl describe clusterrole deployment-clusterrole-yang

Name:         deployment-clusterrole-yang
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources          Non-Resource URLs  Resource Names  Verbs
  ---------          -----------------  --------------  -----
  daemonsets.apps    []                 []              [create]
  deployments.apps   []                 []              [create]
  statefulsets.apps  []                 []              [create]

```
## <div id='3-3'/>3.3. ClusterRoleBinding 확인
```
$ kubectl describe clusterrolebinding deployment-clusterrolebinding-yang

Name:         deployment-clusterrolebinding-yang
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  deployment-clusterrole-yang
Subjects:
  Kind            Name             Namespace
  ----            ----             ---------
  ServiceAccount  cicd-token-yang  api-access-yang

```
## <div id='3-4'/>3.4. 권한 테스트
- kubectl auth can-i 명령어로 ServiceAccount 권한 테스트

```
$ kubectl auth can-i create deployments --as=system:serviceaccount:api-access-yang:cicd-token-yang -n api-access-yang
yes
$ kubectl auth can-i create statefulsets --as=system:serviceaccount:api-access-yang:cicd-token-yang -n api-access-yang
yes
$ kubectl auth can-i create daemonsets --as=system:serviceaccount:api-access-yang:cicd-token-yang -n api-access-yang
yes
```






