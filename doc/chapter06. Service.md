# Service

Pod은 자체 IP를 가지고 다른 Pod과 통신할 수 있지만, 쉽게 사라지고 생성되는 특징 때문에 직접 통신하는 방법은 권장하지 않는다. 
쿠버네티스는 Pod과 직접 통신하는 방법 대신, 별도의 고정된 IP를 가진 서비스를 만들고 그 서비스를 통해 Pod에 접근하는 방식을 사용한다.

이러한 개념은 도커에 익숙할수록 처음 접하면 참 어렵다. 
노출 범위에 따라 CluterIP, NodePort, LoadBalancer 타입으로 나누어져 있다.

#### Service(ClusterIP) 만들기

ClusterIP는 클러스터 내부에 새로운 IP를 할당하고 여러 개의 Pod을 바라보는 로드밸런서 기능을 제공한다. 
그리고 서비스 이름을 내부 도메인 서버에 등록하여 Pod 간에 서비스 이름으로 통신할 수 있다.

    하나의 YAML파일에 여러 개의 리소스를 정의할 땐 "---"를 구분자로 사용합니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: redis
spec:
  selector:
    matchLabels:
      app: counter
      tier: db
  template:
    metadata:
      labels:
        app: counter
        tier: db
    spec:
      containers:
        - name: redis
          image: redis
          ports:
            - containerPort: 6379
              protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  ports:
    - port: 6379
      protocol: TCP
  selector:
    app: counter
    tier: db
```

```shell
kubectl apply -f counter-redis-svc.yml

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/service$ kubectl apply -f counter-redis-svc.yml
deployment.apps/redis created
service/redis created

# Pod, ReplicaSet, Deployment, Service 상태 확인
kubectl get all

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/service$ kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/redis-77b9988d97-r9njm   1/1     Running   0          7s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    40h
service/redis        ClusterIP   10.105.157.31   <none>        6379/TCP   7s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis   1/1     1            1           7s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/redis-77b9988d97   1         1         1       7s
```
redis Deployment와 Service가 생성된 것을 볼 수 있다.

**같은 클러스터에서 생성된 Pod이라면 redis라는 도메인으로 redis Pod에 접근** 할 수 있다 
(redis.default.svc.cluster.local로도 접근가능 합니다. 서로 다른 namespace와 cluster를 구분할 수 있다.)


| 정의                | 설명                |
|-------------------|-------------------|
| spec.ports.port   | 서비스가 생성할 port     |
| spec.ports.targetPort| 서비스가 접근할 Pod의 Port|
| spec.selector | 서비스가 접근할 Pod의 label 조건 |

redis Service의 selector는 redis Deployment에 정의한 label을 사용했기 때문에 해당 Pod을 가리킨다. 
그리고 해당 Pod의 6379 포트로 연결한다.

이제 redis에 접근할 counter 앱을 Deployment로 만든다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: counter
spec:
  selector:
    matchLabels:
      app: counter
      tier: app
  template:
    metadata:
      labels:
        app: counter
        tier: app
    spec:
      containers:
        - name: counter
          image: ghcr.io/subicura/counter:latest
          env:
            - name: REDIS_HOST
              value: "redis"
            - name: REDIS_PORT
              value: "6379"
```
counter app Pod에서 redis Pod으로 접근이 되는지 테스트 해본다
```shell
kubectl apply -f counter-app.yml

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/service$ kubectl apply -f counter-app.yml
deployment.apps/counter created

# counter app에 접근
kubectl get po

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/service$ kubectl get po
NAME                       READY   STATUS    RESTARTS   AGE
counter-6b955fcffc-4mpz9   1/1     Running   0          24s
redis-77b9988d97-r9njm     1/1     Running   0          23m

kubectl exec -it counter-<xxxxx> -- sh

  roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/service$ kubectl exec -it counter-6b955fcffc-4mpz9 -- sh
/app # curl localhost:3000
1
/app # curl localhost:3000
2
/app # telnet redis 6379
Connected to redis
dbsize
:1
keys *
*1
$5
count
get count
$1
2
quit
+OK
Connection closed by foreign host
/app #
```
Service를 통해 Pod과 성공적으로 연결되었다.

#### Service 생성 흐름

Service는 각 Pod를 바라보는 로드밸런서 역할을 하면서 내부 도메인서버에 새로운 도메인을 생성한다.
Service가 어떻게 동작하는지 살펴본다.

* Endpoint Controller는 Service와 Pod을 감시하면서 조건에 맞는 Pod의 IP를 수집
* Endpoint Controller가 수집한 IP를 가지고 Endpoint 생성
* Kube-Proxy는 Endpoint 변화를 감시하고 노드의 iptables을 설정
* CoreDNS는 Service를 감시하고 서비스 이름과 IP를 CoreDNS에 추가

iptables는 커널kernel 레벨의 네트워크 도구이고 CoreDNS는 빠르고 편리하게 사용할 수 있는 클러스터 내부용 도메인 네임 서버이다.. 
각각의 역할은 iptables 설정으로 여러 IP에 트래픽을 전달하고 CoreDNS를 이용하여 IP 대신 도메인 이름을 사용한다.


```shell
kubectl get endpoints
kubectl get ep #줄여서

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/service$ kubectl get ep
NAME         ENDPOINTS           AGE
kubernetes   192.168.49.2:8443   41h
redis        10.244.0.59:6379    37m

# redis Endpoint 확인
kubectl describe ep/redis

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/service$ kubectl describe ep/redis
Name:         redis
Namespace:    default
Labels:       <none>
Annotations:  endpoints.kubernetes.io/last-change-trigger-time: 2023-11-29T00:30:14Z
Subsets:
  Addresses:          10.244.0.59
  NotReadyAddresses:  <none>
  Ports:
    Name     Port  Protocol
    ----     ----  --------
    <unset>  6379  TCP

Events:  <none>
```

Endpoint Addresses 정보에 Redis Pod의 IP가 보인다. (Replicas가 여러개였다면 여러 IP가 보인다.)

### Service(NodePort) 만들기

CluterIP는 클러스터 내부에서만 접근할 수 있습니다. 클러스터 외부(노드)에서 접근할 수 있도록 NodePort 서비스를 만들어본다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: counter-np
spec:
  type: NodePort
  ports:
    - port: 3000
      protocol: TCP
      nodePort: 31000
  selector:
    app: counter
    tier: app
```

```shell
kubectl apply -f counter-nodeport.yml

# 서비스 상태 확인
kubectl get svc

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/service$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
counter-np   NodePort    10.104.4.16     <none>        3000:31000/TCP   12s
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          41h
redis        ClusterIP   10.105.157.31   <none>        6379/TCP         45m
```

minikube ip로 테스트 클러스터의 노드 IP를 구하고 31000으로 접근해본다 

    Docker driver를 사용중이라면 minikube service counter-np 명령어를 이용하여 접속한다.

```shell
curl 192.168.49.2:31000 # 또는 브라우저에서 접근

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/service$ curl 192.168.49.2:31000
3


roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/service$ minikube service counter-np
|-----------|------------|-------------|---------------------------|
| NAMESPACE |    NAME    | TARGET PORT |            URL            |
|-----------|------------|-------------|---------------------------|
| default   | counter-np |        3000 | http://192.168.49.2:31000 |
|-----------|------------|-------------|---------------------------|
🏃  Starting tunnel for service counter-np.
|-----------|------------|-------------|------------------------|
| NAMESPACE |    NAME    | TARGET PORT |          URL           |
|-----------|------------|-------------|------------------------|
| default   | counter-np |             | http://127.0.0.1:46859 |
|-----------|------------|-------------|------------------------|
🎉  Opening service default/counter-np in default browser...
👉  http://127.0.0.1:46859
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.

```
NodePort는 클러스터의 모든 노드에 포트를 오픈한다. 
지금은 테스트라서 하나의 노드밖에 없지만 여러 개의 노드가 있다면 아무 노드로 접근해도 지정한 Pod으로 접근할 수 있다.

#### Service(LoadBalancer) 만들기

자동으로 살아 있는 노드에 접근하기 위해 모든 노드를 바라보는 Load Balancer가 필요하다. 
브라우저는 NodePort에 직접 요청을 보내는 것이 아니라 Load Balancer에 요청하고 Load Balancer가 알아서 살아 있는 노드에 접근하면 NodePort의 단점을 없앨 수 있다.

Load Balancer를 생성

```yaml
apiVersion: v1
kind: Service
metadata:
  name: counter-lb
spec:
  type: LoadBalancer
  ports:
    - port: 30000
      targetPort: 3000
      protocol: TCP
  selector:
    app: counter
    tier: app
```


```shell
kubectl apply -f counter-lb.yml

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/service$ kubectl apply -f counter-lb.yml
service/counter-lb created

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/service$ kubectl get svc
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
counter-lb   LoadBalancer   10.106.64.82    <pending>     30000:31085/TCP   50s
counter-np   NodePort       10.104.4.16     <none>        3000:31000/TCP    52m
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP           42h
redis        ClusterIP      10.105.157.31   <none>        6379/TCP          98m
```

counter-lb가 생성되었지만, EXTERNAL-IP가 <pending>인 것을 확인할 수 있다. 
사실 Load Balancer는 AWS, Google Cloud, Azure 같은 클라우드 환경이 아니면 사용이 제한적이다.

#### miniKube에 가상 LoadBalancer 만들기

Load Balancer를 사용할 수 없는 환경에서 가상 환경을 만들어 주는 것이 MetalLB라는 것이다. 
minikube에서는 현재 떠 있는 노드를 Load Balancer로 설정한다. minikube의 addons 명령어로 활성화한다.

```shell
minikube addons enable metallb

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/service$ minikube addons enable metallb
❗  metallb is a 3rd party addon and is not maintained or verified by minikube maintainers, enable at your own risk.
❗  metallb does not currently have an associated maintainer.
    ▪ Using image quay.io/metallb/speaker:v0.9.6
    ▪ Using image quay.io/metallb/controller:v0.9.6
🌟  The 'metallb' addon is enabled
```
그리고 minikube ip(192.168.49.2)명령어로 확인한 IP를 ConfigMap으로 지정해야 한다.(minikube ip 명령의 값은 환경마다 다름)

minikube를 이용하여 손쉽게 metallb 설정을 할 수 있다.

```shell
minikube addons configure metallb

-- Enter Load Balancer Start IP: # minikube ip 결과값 입력
-- Enter Load Balancer End IP: # minikube ip 결과값 입력
    ▪ Using image metallb/speaker:v0.9.6
    ▪ Using image metallb/controller:v0.9.6
✅  metallb was successfully configured


roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/service$ minikube addons configure metallb
-- Enter Load Balancer Start IP: 192.168.49.2
-- Enter Load Balancer End IP: 192.168.49.2
    ▪ Using image quay.io/metallb/controller:v0.9.6
    ▪ Using image quay.io/metallb/speaker:v0.9.6
✅  metallb was successfully configured

```
minikube를 사용하지 않고 직접 ConfigMap을 작성할 수도 있다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
      - name: default
        protocol: layer2
        addresses:
          - 192.168.49.2/32
```

```shell
kubectl apply -f metallb-cm.yml

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/service$ kubectl apply -f metallb-cm.yml
configmap/config configured

# 다시 서비스 확인
kubectl get svc

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/service$ kubectl get svc
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)           AGE
counter-lb   LoadBalancer   10.106.64.82    192.168.49.2   30000:31085/TCP   21m
counter-np   NodePort       10.104.4.16     <none>         3000:31000/TCP    73m
kubernetes   ClusterIP      10.96.0.1       <none>         443/TCP           42h
redis        ClusterIP      10.105.157.31   <none>         6379/TCP          118m
```
    Docker driver를 사용중이라면 minikube service counter-lb 명령어를 이용하여 접속한다.

```shell
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/service$ minikube service counter-lb
|-----------|------------|-------------|---------------------------|
| NAMESPACE |    NAME    | TARGET PORT |            URL            |
|-----------|------------|-------------|---------------------------|
| default   | counter-lb |       30000 | http://192.168.49.2:31085 |
|-----------|------------|-------------|---------------------------|
🏃  Starting tunnel for service counter-lb.
|-----------|------------|-------------|------------------------|
| NAMESPACE |    NAME    | TARGET PORT |          URL           |
|-----------|------------|-------------|------------------------|
| default   | counter-lb |             | http://127.0.0.1:41189 |
|-----------|------------|-------------|------------------------|
🎉  Opening service default/counter-lb in default browser...
👉  http://127.0.0.1:41189
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.
```

실전에선 NodePort와 LoadBalancer를 제한적으로 사용한다. 
보통 웹 애플리케이션을 배포하면 80 또는 443 포트를 사용하고 하나의 포트에서 여러 개의 서비스를 도메인이나 경로에 따라 다르게 연결하기 때문이다. 
이 부분은 뒤에 Ingress에서 자세히 알아봅니다.


#### 실습

문제1. echo 서비스를 NodePort로 32000 포트로 오픈한다.

| 키 | 값 |
|-----------------|-----------------|
| Deployment 이름| echo |
| Deployment Lable | app: echo|
| Deployment 복제수 | 3|
| Container이름 | echo|
| Container 이미지 | ghrc.io/subicura/echo:v1|
| NodePort 이름 | echo |
| NodePort Port | 3000 | 
| NodePort NodePort | 32000|

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo
  labels:
    app: echo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: echo
  template:
    metadata:
      name: echo
      labels:
        app: echo
    spec:
      containers:
        - name: echo
          image: ghrc.io/subicura/echo:v1
---
apiVersion: v1
kind: Service
metadata:
  name: echo-np
spec:
  type: NodePort
  ports:
    - port: 3000
      protocol: TCP
      nodePort: 32000
  selector:
    app: echo

```