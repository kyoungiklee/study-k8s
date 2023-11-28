# Pod
Pod 는 쿠버네티스에서 관리하는 가장 작은 배포 단위이다. 
쿠버네티스와 도커의 차이점은 도커는 컨테이너를 만들지만 쿠버네티스는 컨테이너 대신 Pod를 만든다. Pod 는 한 개 또는 여러개의 컨테이너를 포함한다.

#### 간단한 Pod 생성
```shell
kubectl run echo --image ghcr.io/subicura/echo:v1

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl run echo --image ghcr.io/subicura/echo:v1
pod/echo created

# Pod 목록 조회
kubectl get pod

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl get pod/echo
NAME   READY   STATUS    RESTARTS   AGE
echo   1/1     Running   0          49s

```

생성된 Pod의 상태를 간략하게 확인할 수 있다. 상태는 컨테이너가 정상적으로 생성되면 running 으로 바뀌고 오류가 있다면 에러 상태를 표시한다.

```shell
# 단일 Pod 상세 확인
kubectl describe pod/echo

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl describe pod/echo
Name:             echo
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Tue, 28 Nov 2023 14:03:17 +0900
Labels:           run=echo
Annotations:      <none>
Status:           Running
IP:               10.244.0.16
IPs:
  IP:  10.244.0.16
Containers:
  echo:
    Container ID:   docker://1876df415d706802891087968c7c035f922de0cc5458d1b25854b42618681544
    Image:          ghcr.io/subicura/echo:v1
    Image ID:       docker-pullable://ghcr.io/subicura/echo@sha256:486ea26869d277e4090812fe5eebb694eb754791bb5568f23983775948eee3c3
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 28 Nov 2023 14:03:17 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-nqdxs (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-nqdxs:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  3m3s  default-scheduler  Successfully assigned default/echo to minikube
  Normal  Pulled     3m3s  kubelet            Container image "ghcr.io/subicura/echo:v1" already present on machine
  Normal  Created    3m3s  kubelet            Created container echo
  Normal  Started    3m3s  kubelet            Started container echo

```
describe 명령어는 해당 리소스의 상세한 정보를 알려줍니다. 쿠버네티스를 운영하면서 가장 많이 확인하는 부분은 Events입니다. 현재 Pod의 상태를 이벤트별로 확인할 수 있습니다


#### Pod 생성 분석

Pod는 minikube 클러스터 안에 Pod가 있고 Pod 안에 컨테이너가 있다.

kubectl run 을 실행하고 Pod가 생성되는 과장
 * Scheduler는 API서버를 감시하면서 할당되지 않은(unassigned Pod)이 있는지 체크
 * Scheduler는 할당되지 않은 Pod을 감지하고 적절한 노드(node)에 할당 (minikube는 단일 노드)
 * 노드에 설치된 kubelet은 자신의 노드에 할당된 Pod이 있는지 체크
 * kubelet은 Scheduler에 의해 자신에게 할당된 Pod의 정보를 확인하고 컨테이너 생성
 * kubelet은 자신에게 할당된 Pod의 상태를 API 서버에 전달


#### Pod 제거

```shell
kubectl delete pod/echo

roadseeker@DESKTOP-ARE0NV9:~$ kubectl delete pod/echo
pod "echo" deleted

```

#### YAML로 설정파일(Spec) 작성하기
kubectl run 명령어는 실전에선 거의 사용하지 않는다.  훨씬 더 복잡하고 다양한 설정이 필요한데 이를 kubectl 명령어로 표현하면 금방 복잡해지고 관리가 어렵다
원하는 리소스를 YAML 파일로 작성하면 복잡한 내용을 표현하기 좋고 변경된 내용을 버전 관리할 수 있다.

위에서 만든 Pod을 YAML 파일로 정의한다.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: echo
  labels:
    app: echo
spec:
  containers:
    - name: app
      image: ghcr.io/subicura/echo:v1
```
* version : 오브젝트 버전 - v1, app/v1, networking.k8s.io/v1, ....
* kind : 종류 - Pod, ReplicaSet, Deployment, Service, ....
* metadata : 메타데이터 -  name, label, annotation 으로 구성
* spec : 상세설명 - 리소스 종류마다 다름
  
version, kind, metadata, spec는 리소스를 정의할 때 반드시 필요한 요소입니다

````shell
# Pod 생성
kubectl apply -f echo-pod.yml

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$ kubectl apply -f echo-pod.yml
pod/echo created

# Pod 목록 조회
kubectl get pod

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$ kubectl get pod
NAME   READY   STATUS    RESTARTS   AGE
echo   1/1     Running   0          20s
# Pod 로그 확인
kubectl logs echo
kubectl logs -f echo

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$ kubectl logs echo
{"level":30,"time":1701149472801,"pid":7,"hostname":"echo","msg":"Server listening at http://0.0.0.0:3000"}
{"level":30,"time":1701149472802,"pid":7,"hostname":"echo","msg":"server listening on http://0.0.0.0:3000"}

# Pod 컨테이너 접속
kubectl exec -it echo -- sh
# ls
# ps
# exit

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$ kubectl exec -it echo -- sh
/app # ls
Dockerfile         README.md          app.js             node_modules       package-lock.json  package.json
/app # ps
PID   USER     TIME  COMMAND
    1 root      0:00 /sbin/tini -- node app.js
    7 root      0:00 node app.js
   20 root      0:00 sh
   27 root      0:00 ps
/app # exit
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$

# Pod 제거
kubectl delete -f echo-pod.yml

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$ kubectl delete -f echo-pod.yml
pod "echo" deleted
````

컨테이너 생성과 실제 서비스 준비는 약간의 차이가 있습니다. 서버를 실행하면 바로 접속할 수 없고 짧게는 수초, 길게는 수분의 초기화 시간이 필요하다
제로 접속이 가능할 때 서비스가 준비되었다고 말할 수 있습니다.
쿠버네티스는 컨테이너가 생성되고 서비스가 준비되었다는 것을 체크하는 옵션을 제공하여 초기화하는 동안 서비스되는 것을 막을 수 있다.

#### livenessProbe
컨테이너가 정상적으로 동작하는지 체크하고 정상적으로 동작하지 않는다면 컨테이너를 재시작하여 문제를 해결한다.
정상이라는 것은 여러 가지 방식으로 체크할 수 있는데 여기서는 http get 요청을 보내 확인하는 방법을 사용한다.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: echo-ip
  labels:
    app: echo
spec:
  containers:
    - name: app
      image: ghcr.io/subicura/echo:v1
      livenessProve:
        httpGet:
          path: /not/exist
          port: 8080
        initialDelaySeconds: 5
        timeoutSeconds: 2 #Default 1
        periodSeconds: 5 #Default 10
        failureThreshold: 1 #Default 3
```

```shell
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$ kubectl apply -f echo-rp.yml
pod/echo-rp created


roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$ kubectl get pod
NAME      READY   STATUS    RESTARTS     AGE
echo-rp   1/1     Running   2 (2s ago)   23s
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$ kubectl get pod
NAME      READY   STATUS    RESTARTS      AGE
echo-rp   1/1     Running   3 (11s ago)   41s
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$ kubectl get pod
NAME      READY   STATUS             RESTARTS     AGE
echo-rp   0/1     CrashLoopBackOff   3 (4s ago)   45s
```
#### 다중 컨테이너

대부분 1 Pod = 1 컨테이너지만 여러 개의 컨테이너를 가진 경우도 꽤 흔하다.
하나의 Pod에 속한 컨테이너는 서로 네트워크를 localhost로 공유하고 동일한 디렉토리를 공유할 수 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: counter
  labels:
    app: counter
spec:
  containers:
    - name: app
      image: ghcr.io/subicura/counter:latest
      env:
        - name: REDIS_HOST
          value: "localhost"
    - name: db
      image: redis
```

요청횟수를 redis에 저장하는 간단한 웹 애플리케이션을 다중 컨테이너로 생성한다.
같은 Pod에 컨테이너가 생성되었기 때문에 counter 앱은 redis 를 localhost로 접근할 수 있다. 

```shell
# Pod 생성
kubectl apply -f counter-pod-redis.yml

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$ kubectl apply -f counter-pod-redis.yml
pod/counter created

# Pod 목록 조회
kubectl get pod

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$ kubectl get pod
NAME      READY   STATUS    RESTARTS   AGE
counter   2/2     Running   0          56s

# Pod 로그 확인
kubectl logs counter # 오류 발생 (컨테이너 지정 필요)
kubectl logs counter app
kubectl logs counter db

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/pod$ kubectl logs counter db
1:C 28 Nov 2023 06:12:09.712 * oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 28 Nov 2023 06:12:09.712 * Redis version=7.2.3, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 28 Nov 2023 06:12:09.712 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
1:M 28 Nov 2023 06:12:09.714 * monotonic clock: POSIX clock_gettime
1:M 28 Nov 2023 06:12:09.715 * Running mode=standalone, port=6379.
1:M 28 Nov 2023 06:12:09.717 * Server initialized
1:M 28 Nov 2023 06:12:09.717 * Ready to accept connections tcp

# Pod의 app컨테이너 접속
kubectl exec -it counter -c app -- sh
# curl localhost:3000
# curl localhost:3000
# telnet localhost 6379
  dbsize
  KEYS *
  GET count
  quit

# Pod 제거
kubectl delete -f counter-pod-redis.yml
```
멀티 컨테이너는 도커에선 볼 수 없던 개념입니다. 쿠버네티스는 멀티 컨테이너를 이용한 다양한 패턴이 존재합니다. 로그를 수집하는 별도의 컨테이너를 같은 Pod으로 배포한다던가, 서버가 실행되기 전 데이터베이스를 마이그레이션 하는 초기화 컨테이너를 만들 수도 있다

#### 마무리
Pod은 쿠버네티스에서 굉장히 중요한 요소이지만 단독으로 사용하는 경우는 거의 없습니다. Pod이 컨테이너를 관리하듯이 다른 컨트롤러가 Pod을 관리한다.

실습 1 - mongodb pod 생성
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: mongodb
    labels:
      app: mongo
spec:
  containers:
    - name: mongodb
      image: mongo:4
  

```

실습 2 mariadb pod 생성
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mariadb
  labels:
    app: mariadb
spec:
  containers:
    - name: mariadb
      image: mariadb:10.7
      env:
        - name: MYSQL_ROOT_PASSWORD
          value: "123456"
```