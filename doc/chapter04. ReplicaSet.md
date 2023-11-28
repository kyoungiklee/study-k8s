# REplicaSet

Pod을 단독으로 만들면 Pod에 어떤 문제가 생겼을 때 자동으로 복구되지 않는다. 
이러한 Pod을 정해진 수만큼 복제하고 관리하는 것이 ReplicaSet 이다.

#### ReplicaSet 만들기
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: echo-rs
spec:
  replicas: 1
  selector:
    matchLabels:
      app: echo
      tier: app
  template:
    metadata:
      labels:
        app: echo
        tier: app
    spec:
      containers:
        - name: echo
          image: ghcr.io/subicura/echo:v1

```

ReplicaSet 생성
```shell
# ReplicaSet 생성
kubectl apply -f echo-rs.yml

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/replicaset$ kubectl apply -f echo-rs.yml
replicaset.apps/echo-rs created

# 리소스 확인
kubectl get po,rs

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$ kubectl get po,rs
NAME                READY   STATUS    RESTARTS   AGE
pod/echo-rs-bklj5   1/1     Running   0          103s

NAME                      DESIRED   CURRENT   READY   AGE
replicaset.apps/echo-rs   1         1         1       103s

kubectl
```
ReplicaSet과 Pod이 같이 생성된 것을 볼 수 있다.
ReplicaSet은 label을 체크해서 원하는 수의 Pod이 없으면 새로운 Pod을 생성한다.
* spec.selector - label 체크 조건
* spec.replicas - 원하는 pod 갯수
* spec.template - 생성할 pod 명세

template을 자세히 보니 이전에 본 Pod 설정 파일과 완전히 동일한 것을 알 수 있다.

생성된 Pod 의 label 확인
```shell
kubectl get pod --show-labels

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$ kubectl get pod --show-labels
NAME            READY   STATUS    RESTARTS   AGE    LABELS
echo-rs-bklj5   1/1     Running   0          7m8s   app=echo,tier=app
```

임의로 label 제거시
```shell
# app- 를 지정하면 app label을 제거
kubectl label pod/echo-rs-bklj5 app- #환경마다 다름

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$ kubectl label pod/echo-rs-bklj5 app-
pod/echo-rs-bklj5 unlabeled

# 다시 Pod 확인
kubectl get pod --show-labels

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$ kubectl get pod --show-labels
NAME            READY   STATUS    RESTARTS   AGE   LABELS
echo-rs-bklj5   1/1     Running   0          11m   tier=app
echo-rs-c8qkm   1/1     Running   0          49s   app=echo,tier=app
```
기존에 생성된 Pod의 app label이 사라지면서 selector에 정의한 app=echo,tier=app 조건을 만족하는 Pod의 개수가 0이 되어 새로운 Pod이 만들어졌다.


다시 label 추가
```shell
# app- 를 지정하면 app label을 제거
kubectl label pod/echo-rs-tcdwj app=echo

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$ kubectl label pod/echo-rs-bklj5 app=echo
pod/echo-rs-bklj5 labeled

# 다시 Pod 확인
kubectl get pod --show-labels

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$ kubectl get pod --show-labels
NAME            READY   STATUS    RESTARTS   AGE   LABELS
echo-rs-bklj5   1/1     Running   0          15m   app=echo,tier=app
```
replicas에 정의한 대로 Pod의 개수를 1로 유지하기 위해 기존 Pod을 제거합니다.

ReplicaSet 이 작동하는 방식
* ReplicaSet Controller는 ReplicaSet조건을 감시하면서 현재 상태와 원하는 상태가 다른 것을 체크
* ReplicaSet Controller가 원하는 상태가 되도록 Pod을 생성하거나 제거
* Scheduler는 API서버를 감시하면서 할당되지 않은 Pod이 있는지 체크
* Scheduler는 할당되지 않은 새로운 Pod을 감지하고 적절한 노드에 배치
* 이후 노드는 기존대로 동작

ReplicaSet은 ReplicaSet Controller가 관리하고 Pod의 할당은 여전히 Scheduler가 관리한다.

#### 스케일 아웃
ReplicaSet을 이용하면 손쉽게 Pod을 여러개로 복제할 수 있다.
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: echo-rs
spec:
  replicas: 4
  selector:
    matchLabels:
      app: echo
      tier: app
  template:
    metadata:
      labels:
        app: echo
        tier: app
    spec:
      containers:
        - name: echo
          image: ghcr.io/subicura/echo:v1
```
```shell
kubectl apply -f echo-rs-scaled.yml

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/replicaset$ kubectl apply -f echo-rs-scaled.yml
replicaset.apps/echo-rs created

# Pod 확인
kubectl get pod,rs


roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/replicaset$ kubectl get pod,rs
NAME                READY   STATUS    RESTARTS   AGE
pod/echo-rs-5bcwh   1/1     Running   0          100s
pod/echo-rs-6dksp   1/1     Running   0          100s
pod/echo-rs-6pbrd   1/1     Running   0          100s
pod/echo-rs-krblr   1/1     Running   0          100s

NAME                      DESIRED   CURRENT   READY   AGE
replicaset.apps/echo-rs   4         4         4       100s
```

#### 마무리
ReplicaSet은 원하는 개수의 Pod을 유지하는 역할을 담당한다. 
label을 이용하여 Pod을 체크하기 때문에 label이 겹치지 않게 신경써서 정의해야 한다.

#### 실습
| 키                   | 값            |
|---------------------|--------------|
| ReplicaSet 이름       | nginx        |
| ReplicaSet selector | app:nginx    |
| ReplicaSet 복제수      | 3            |
| Container 이름        | nginx        |
| Container 이미지       | nginx:latest |

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
spec:
  replicas: 3
  selector: 
    matchLabels:  
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
  spec:
    containers:
      - name: nginx
        image: nginx:latest
```

```shell
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/replicaset$ kubectl get po,rs
NAME              READY   STATUS              RESTARTS   AGE
pod/nginx-9j4k5   1/1     Running             0          12s
pod/nginx-k5slz   1/1     Running             0          12s
pod/nginx-vpq7r   0/1     ContainerCreating   0          12s

NAME                    DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx   3         3         2       12s
```
```shell
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/replicaset$ kubectl delete -f example1.yml
replicaset.apps "nginx" deleted
```
