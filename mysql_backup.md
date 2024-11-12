# kubernetes mysql 구축 (pv,pvc 연결) 및 백업 파일 생성 가이드

## 목차
1. [문서 개요](#1)
    * [1.1. 목적](#1-1)
2. [pv, pvc 생성](#2)
    * [2.1. pv 생성 파일 작성](#2-1)
    * [2.2. pv 생성 결과 확인](#2-2)
    * [2.3. pvc 생성 파일 작성](#2-3)
    * [2.4. pvc 생성 결과 확인](#2-4)
    * [2.5. pv, pvc 연결 확인](#2-5)
3. [configmap 생성](#3)
    * [3.1. configmap 생성 파일 작성](#3-1)
    * [3.2. configmap 생성 결과 확인](#3-2)
4. [secret 생성](#4)
    * [4.1. secret 생성 파일 작성](#4-1)
    * [4.2. secret 생성 결과 확인](#4-2)
5. [statefulSet 생성](#5)
    * [5.1. statefulSet 생성 파일 작성](#5-1)
    * [5.2. statefulSet 생성 결과 확인](#5-2)
6. [service 생성](#6)
    * [6.1. service 생성 파일 작성](#6-1)
    * [6.2. service 생성 결과 확인](#6-2)
7. [mysql 접속 확인](#7)
    * [7.1. 접속 방법](#7-1)
    * [7.1.1. 외부에서 접속 방법](#7-1-1)
    * [7.1.2. 내부에서 접속 방법](#7-1-2)
    * [7.2. 데이터 확인 ](#7-2)
8. [cronjob 생성](#8)
    * [8.1. cronjob 생성 파일 작성](#8-1)
    * [8.2. cronjob 실행 테스트](#8-2)


# <div id='1'/> 1. 문서 개요

## <div id='1-1'/> 1.1. 목적
본 문서는 쿠버네티스 환경에 mysql 서비스를 배포하는 방법과 주기적으로 백업시키는 방법에 대해 기술한다.



# <div id='2'/> 2. pv, pvc 생성

## <div id='2-1'/>2.1. pv 생성 파일 작성

k-paas 전문가 교육 실습에서 mysql 파드가 재배포되면 내부에 넣어두었던 테이블과 데이터들이 초기화된 것을 볼수 있음
이는 pod가 삭제되면서 볼륨또한 함꼐 삭제되었기 때문

이러한 데이터 유실을 막기 위해 pv,pvc를 붙임
붙인 다음에 pod를 지우고 다시 배포하게되면 데이터가 그대로 남아있음을 확인할 수 있음
<br>
vi mysql-pv.yaml

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv-volume # pv 이름
  labels:
    type: local
spec:
  storageClassName: cp-storageclass # 사용할 storageclass명
  capacity:
    storage: 10Gi # 스토리지 용량 크기
  accessModes:
    - ReadWriteOnce # 하나의 Pod에서만 access가 가능하도록 설정, ReadWriteMany는 여러 개 노드에서 접
근 가능
  hostPath:
    path: "/data/k8s/db/" # node에 저장될 스토리지 공간 

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: backup-pv-volume # pv 이름
  labels:
    type: local
spec:
  storageClassName: cp-storageclass
  capacity:
    storage: 10Gi # 스토리지 용량 크기
  accessModes:
    - ReadWriteOnce # 하나의 Pod에서만 access가 가능하도록 설정, ReadWriteMany는 여러 개 노드에서 접
근 가능
  hostPath:
    path: "/backup/" # node에 저장될 스토리지 공간

```


- storageClassName에는 내가 사용할 storageClass이름을 넣는다 
kubectl get sc -A --> 확인 후 사용할 storageClass를 작성

- accessModes 접근 모드 설정
<br>
*접근 모드 : https://kubernetes.io/ko/docs/concepts/storage/persistent-volumes/#%EC%A0%91%EA%B7%BC-%EB%AA%A8%EB%93%9C

- path : path에 지정된 경로에 파일을 저장함 (해당 경로는 따로 생성해주어야함)
```
sudo mkdir -p /data/k8s/db
sudo mkdir -p /backup/
```

- 명령어 실행 결과

```kubectl apply -f mysql-pv.yaml
persistentvolume/mysql-pv-volume created
persistentvolume/backup-pv-volume created
```




## <div id='2-2'/>2.2. pv 생성 결과 확인
```
ubuntu@ta-task-cluster-1:~/edu$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS            CLAIM                                        STORAGECLASS      REASON   AGE
backup-pv-volume                           10Gi       RWO            Retain           Available                                                cp-storageclass            23s
mysql-pv-volume                            10Gi       RWO            Retain           Available                                                cp-storageclass            23s
```
- claim을 아직 생성 및 연결 하지 않았기 때문에 STATUS가 Available, claim에 공백

## <div id='2-3'/>2.3. pvc 생성 파일 작성
vi mysql-pvc.yaml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim # pvc 이름
spec:
  storageClassName: cp-storageclass
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: backup-pv-claim # pvc 이름
spec:
  storageClassName: cp-storageclass
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

- pvc의 accessMode는 사용하고자 하는 PV의 accessModes와 동일한 옵션을 사용해야 bound 가능
- 순차적 매핑으로 pv 생성 순서와 동일하게 pvc도 작성

```
ubuntu@ta-task-cluster-1:~/edu$ kubectl apply -f mysql-pvc.yaml 
persistentvolumeclaim/mysql-pv-claim created
persistentvolumeclaim/backup-pv-claim created
```

## <div id='2-4'/>2.4. pvc 생성 결과 확인
```
ubuntu@ta-task-cluster-1:~/edu$ kubectl get pvc
NAME              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
backup-pv-claim   Bound    backup-pv-volume                           10Gi       RWO            cp-storageclass   22s
mysql-pv-claim    Bound    mysql-pv-volume                            10Gi       RWO            cp-storageclass   22s

```
## <div id='2-5'/>2.5. pv, pvc 연결 확인
```
ubuntu@ta-task-cluster-1:~/edu$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                                        STORAGECLASS      REASON   AGE
backup-pv-volume                           10Gi       RWO            Retain           Bound      default/backup-pv-claim                      cp-storageclass            4m37s
mysql-pv-volume                            10Gi       RWO            Retain           Bound      default/mysql-pv-claim                       cp-storageclass            4m37s
```
- pv, pvc가 연결된 것을 확인 (STATUS: Bound 상태)



# <div id='3'/> 3. configmap 생성

## <div id='3-1'/>3.1. configmap 생성 파일 작성
vi mysql-configmap.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-initdb-config
data:
  master.cnf: |
    [mysqld]
    log-bin
    character-set-server=utf8mb4
    collation-server=utf8mb4_general_ci
    default-time-zone=Asia/Seoul
  slave.cnf: |
    [mysqld]
    super-read-only
    character-set-server=utf8mb4
    collation-server=utf8mb4_general_ci
    default-time-zone=Asia/Seoul
  init.sql: |
    grant all privileges on *.* to username@localhost identified by 'password';
    grant all privileges on *.* to username@'127.0.0.1' identified by 'password';
    CREATE DATABASE IF NOT EXISTS mydata;
    USE mydata;
    CREATE TABLE friends (id INT, name VARCHAR(256), age INT, gender VARCHAR(3));
    INSERT INTO friends VALUES (1, 'Eric', 30, 'm');
    INSERT INTO friends VALUES (2, 'Luna', 26, 'f');
    INSERT INTO friends VALUES (3, 'Ash', 22, 'm');
```
- mysql 초기 설정을 위해 configmap 생성
- master.cnf  
    [mysqld] : mysql binary logging 기능 활성화
    character-set-server=utf8mb4 : mysql 기본 문자 세트 설정
    collation-server=utf8mb4_general_ci : 서버 기본 정렬 방식 설정
    default-time-zone=Asia/Seoul : 서버 기본 시간대 설정
- slave.cnf:
    super-read-only : slave 서버에서 DB를 읽기 전용으로 사용/master 서버에서 동기화된 데이터 안전하게 유지
- init.sql: mysql서버가 시작될 때 실행할 초기화 sql 스크립트

```
ubuntu@ta-task-cluster-1:~/edu$ kubectl apply -f mysql-configmap.yaml 
configmap/mysql-initdb-config created
```


## <div id='3-2'/>3.2. configmap 생성 결과 확인
```
ubuntu@ta-task-cluster-1:~/edu$ kubectl get configmaps 
NAME                        DATA   AGE
mysql-initdb-config         3      25s
```

# <div id='4'/> 4. secret 생성

## <div id='4-1'/>4.1. secret 생성 파일 작성

vi mysql-secret.yaml
```
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  password: cGFzc3dvcmQ= #password

```
- password에는 base64로 인코딩하여 값을 넣는다<br>
`echo -n 'password' | base64`<br>
`cGFzc3dvcmQ=`

```
ubuntu@ta-task-cluster-1:~/edu$ kubectl apply -f mysql-secret.yaml 
secret/mysql-secret created
```
## <div id='4-2'/>4.2. secret 생성 결과 확인
```
ubuntu@ta-task-cluster-1:~/edu$ kubectl get secret
NAME                              TYPE                 DATA   AGE
mysql-secret                      Opaque               1      30s
```

# <div id='5'/> 5. statefulSet 생성

## <div id='5-1'/>5.1. statefulSet 생성 파일 작성

vi mysql-statefulset.yaml
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  serviceName: mysql
  replicas: 1 # 원하는 복제본 수
  template:
    metadata:
      labels:
        app: mysql
    spec:
       containers:
        - image: mysql:5.6 
          name: mysql
          ports:
            - containerPort: 3306 # Container 포트
          name: mysql
          volumeMounts:
          - name: mysql-persistent-storage
            mountPath: /docker-entrypoint-initdb.d # 해당 폴더에 .sql 파일 존재 시 Container 생성 시 실행
          - name: mysql-data
            mountPath:  /var/lib/mysql
            subPath: "mysql"
          env:
          - name: MYSQL_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysql-secret # Secret의 이름
                key: password # Secret의 data에 들어간 key:value
       volumes:
       - name: mysql-persistent-storage
         configMap:
          name: mysql-initdb-config # configMap 설정
       - name: mysql-data
         persistentVolumeClaim:
          claimName: mysql-pv-claim # pv 볼륨 설정
       - name: backup-data
         persistentVolumeClaim:
          claimName: backup-pv-claim

```

```
ubuntu@ta-task-cluster-1:~/edu$ kubectl apply -f mysql-statefulset.yaml 
Warning: would violate PodSecurity "restricted:v1.28": allowPrivilegeEscalation != false (container "mysql" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "mysql" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "mysql" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "mysql" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
statefulset.apps/mysql created
```
## <div id='5-2'/>5.2. statefulSet 생성 결과 확인
```
ubuntu@ta-task-cluster-1:~/edu$ kubectl get statefulsets.apps 
NAME                            READY   AGE
mysql                           1/1     31s
```
```
ubuntu@ta-task-cluster-1:~/edu$ kubectl get pod
NAME                                  READY   STATUS    RESTARTS        AGE
mysql-0                               1/1     Running   0               50s
````

# <div id='6'/> 6. service 생성

## <div id='6-1'/>6.1. service 생성 파일 작성

vi mysql-svc.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  selector:
    app: mysql
  ports:
  - name: mysql
    port: 3306
    targetPort: 3306
    nodePort: 30044
  type: NodePort
```

```
ubuntu@ta-task-cluster-1:~/edu$ kubectl apply -f mysql-svc.yaml 
service/mysql created
```
## <div id='6-2'/>6.2. service 생성 결과 확인

```
ubuntu@ta-task-cluster-1:~/edu$ kubectl get svc
NAME                                      TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                      AGE
mysql                                     NodePort       10.233.12.52    <none>          3306:30044/TCP               14s
```
# <div id='7'/> 7. mysql 접속 확인

## <div id='7-1'/>7.1. 접속 방법

### <div id='7-1-1'/>7.1.1 외부에서 접속 방법

- 파드 외부에서 접속 방법
```
ubuntu@ta-task-cluster-1:~/edu$ mysql -h [masternode IP] -P 30044 -u root -p

```
### <div id='7-1-2'/>7.1.2 내부에서 접속 방법
- 파드 내부에서 접속 방법
```
ubuntu@ta-task-cluster-1:~/edu$ kubectl exec -it mysql-0 -- /bin/bash

root@mysql-0:/# mysql -u root -p

```



## <div id='7-2'/>7.2. 데이터 확인
- init.sql 데이터 확인
```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mydata             |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.00 sec)

mysql> use mydata;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

mysql> show tables;
+------------------+
| Tables_in_mydata |
+------------------+
| friends          |
+------------------+
1 row in set (0.00 sec)

mysql> select * from  friends;
+------+------+------+--------+
| id   | name | age  | gender |
+------+------+------+--------+
|    1 | Eric |   30 | m      |
|    2 | Luna |   26 | f      |
|    3 | Ash  |   22 | m      |
+------+------+------+--------+

```
>> 초기 데이터가 들어와있는 것을 확인
`INSERT INTO friends VALUES (100, 'test', 24, 'm');`
여기에 테스트 데이터를 넣고 pod나 statefulset을 삭제후 다시 배포하여도 데이터가 남아있는지 확인하기

**명령어 참고**
`kubectl delete -f mysql-statefulset.yaml`


# <div id='8'/> 8. cronjob 생성

## <div id='8-1'/>8.1. cronjob 생성 파일 작성
vi mysql-cronjob.yaml 
```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: mysql-backup-cron
spec:
  schedule: "0 17 * * 5" #매주 금요일 오후 5시 실행
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: mysql-yang
            image: mysql:5.6
            command: ["/bin/bash"]
            args: ["-c", "mysqldump -h [masternode IP] -P 30044 -u root -p$MYSQL_ROOT_PASSWORD --all-databases --skip-lock-tables > /backup/backup-$(date +'%Y%m%d-%H%M%S').sql"]
            env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: password
            - name: TZ
              value: Asia/Seoul #파일저장할때 시간을 서울 기준으로
            volumeMounts:
            - name: backup-storage
              mountPath: /backup
          volumes:
          - name: backup-storage
            persistentVolumeClaim:
              claimName: backup-pv-claim
          restartPolicy: Never
  successfulJobsHistoryLimit: 0
  failedJobsHistoryLimit: 1

```
```
kubectl apply -f mysql-cronjob.yaml 
Warning: would violate PodSecurity "restricted:v1.28": allowPrivilegeEscalation != false (container "mysql-yang" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "mysql-yang" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "mysql-yang" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "mysql-yang" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
cronjob.batch/mysql-backup-cron created
```


```
ubuntu@ta-task-cluster-1:~/edu$ kubectl get cronjobs.batch 
NAME                SCHEDULE     SUSPEND   ACTIVE   LAST SCHEDULE   AGE
mysql-backup-cron   0 17 * * 5   False     0        <none>          41s
```
## <div id='8-2'/>8.2. cronjob 실행 테스트
제대로 된 cronjob이 만들어졌는지를 테스트하려면 cronjob으로부터 job을 만들어보면 된다. 
결국 cronjob이란 것은 정기적으로 job을 만드는 작업이니까!

**명령어 참고**
`kubectl create job --from=cronjob/mysql-backup-cron mysql-backup`

```
ubuntu@ta-task-cluster-1:~/edu$ kubectl get job
NAME           COMPLETIONS   DURATION   AGE
mysql-backup   1/1           6s         6s


ubuntu@ta-task-cluster-1:~/edu$ kubectl get pod
NAME                                  READY   STATUS      RESTARTS        AGE
mysql-backup-bkd4r                    0/1     Completed   0               27s
```

- successfulJobsHistoryLimit 옵션을 0으로 설정해두었기 때문에 자동으로 삭제됨 
성공 파드를 몇개 남겨 둘 것인가
- failedJobsHistoryLimit :실패 파드를 몇개 남겨둘것인가 (나는 1개로 지정)

```
ubuntu@ta-task-cluster-1:~$ cd ../../backup/
ubuntu@ta-task-cluster-1:/backup$ ll
total 3336
drwxr-xr-x  2 root root    4096 Nov 12 10:53 ./
drwxr-xr-x 22 root root    4096 Nov 11 16:31 ../
-rw-r--r--  1 root root 3406146 Nov 12 10:52 backup-20241112-105243.sql
```
현재시간으로 백업 스크립트가 생성된 것을 확인 가능함
