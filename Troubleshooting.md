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

### 1-2. kubectl 명령을 사용하여 POD중 CPU를 가장 많이 사용하는 순서대로 출력

### 1-3. kubectl 명령을 사용하여 NODE중 메모리를 가장 적게 사용하는 순서대로 출력

### 1-4. kubectl 명령을 사용하여 클러스터에 구성된 모든 PV를 capacity별로 sort하여 출력 (json 포멧 활용)

### 1-5. kubectl 명령을 사용하여 클러스터 내 POD의 전체 레이블을 확인 후 배포된 POD중 레이블을 임의로 선택하고, 선택한 레이블을 사용하는 Pod들 중 CPU 소비율이 가장 높은 Pod의 이름을 찾아서 출력