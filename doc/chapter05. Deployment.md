# Deployment

Deployment는 쿠버네티스에서 가장 널리 사용되는 오브젝트입니다. 
ReplicaSet을 이용하여 Pod을 업데이트하고 이력을 관리하여 롤백하거나 특정 버전으로 돌아갈 수 있습니다.

#### Deployment 만들기

이전에 만든 ReplicaSet을 Deployment로 만들어 본다
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-deploy
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

Deployment 실행

```shell
# Deployment 생성
kubectl apply -f echo-deployment.yml

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/deployment$ kubectl apply -f echo-deployment.yml
deployment.apps/echo-deploy created

# 리소스 확인
kubectl get po,rs,deploy

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/deployment$ kubectl get po,rs,deploy
NAME                               READY   STATUS    RESTARTS   AGE
pod/echo-deploy-5b6d56b5d4-cxfbl   1/1     Running   0          70s
pod/echo-deploy-5b6d56b5d4-hwhxn   1/1     Running   0          70s
pod/echo-deploy-5b6d56b5d4-qz9kx   1/1     Running   0          70s
pod/echo-deploy-5b6d56b5d4-vmvxj   1/1     Running   0          70s

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/echo-deploy-5b6d56b5d4   4         4         4       70s

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/echo-deploy   4/4     4            4           70s
```

결과 또한 ReplicaSet과 비슷해 보이지만 Deployment의 진가는 Pod을 새로운 이미지로 업데이트할 때 이다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-deploy
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
          image: ghcr.io/subicura/echo:v2
```

```shell
# 새로운 이미지 업데이트
kubectl apply -f echo-deployment-v2.yml

# 리소스 확인
kubectl get po,rs,deploy
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/deployment$ kubectl get po,rs,deploy
NAME                               READY   STATUS    RESTARTS   AGE
pod/echo-deploy-7b47dfc99f-4dvkx   1/1     Running   0          38s
pod/echo-deploy-7b47dfc99f-58vsv   1/1     Running   0          33s
pod/echo-deploy-7b47dfc99f-8v5q2   1/1     Running   0          32s
pod/echo-deploy-7b47dfc99f-mlt9v   1/1     Running   0          38s

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/echo-deploy-5b6d56b5d4   0         0         0       34m
replicaset.apps/echo-deploy-7b47dfc99f   4         4         4       38s

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/echo-deploy   4/4     4            4           34m
```

```shell
NAME                               READY   STATUS    RESTARTS   AGE
pod/echo-deploy-5b6d56b5d4-cxfbl   1/1     Running   0          70s
pod/echo-deploy-5b6d56b5d4-hwhxn   1/1     Running   0          70s
pod/echo-deploy-5b6d56b5d4-qz9kx   1/1     Running   0          70s
pod/echo-deploy-5b6d56b5d4-vmvxj   1/1     Running   0          70s
# 아래와 같이 변경됨
NAME                               READY   STATUS    RESTARTS   AGE
pod/echo-deploy-7b47dfc99f-4dvkx   1/1     Running   0          38s
pod/echo-deploy-7b47dfc99f-58vsv   1/1     Running   0          33s
pod/echo-deploy-7b47dfc99f-8v5q2   1/1     Running   0          32s
pod/echo-deploy-7b47dfc99f-mlt9v   1/1     Running   0          38s
```
Pod가 모두 새로운 버전으로 변경되었다.

Deployment는 새로운 이미지로 업데이트하기 위해 ReplicaSet을 이용한다. 버전을 업데이트하면 새로운 ReplicaSet을 생성하고 해당 ReplicaSet이 새로운 버전의 Pod을 생성한다.

* 새로운 ReplicaSet을 0 -> 1개로 조정하고 정상적으로 Pod이 동작하면 기존 ReplicaSet을 4 -> 3개로 조정한다.
* 새로운 ReplicaSet을 1 -> 2개로 조정하고 정상적으로 Pod이 동작하면 기존 ReplicaSet을 3 -> 2개로 조정한다.
* 새로운 ReplicaSet을 2 -> 3개로 조정하고 정상적으로 Pod이 동작하면 기존 ReplicaSet을 2 -> 1개로 조정한다.
* 최종적으로 새로운 ReplicaSet을 4개로 조정하고 정상적으로 Pod이 동작하면 기존 ReplicaSet을 0개로 조정한다.
* 업데이트 완료

Deployment의 상세 상태를 보면 더 자세히 알 수 있다.
```shell
kubectl describe deploy/echo-deploy

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/deployment$ kubectl describe deploy/echo-deploy
Name:                   echo-deploy
Namespace:              default
CreationTimestamp:      Tue, 28 Nov 2023 17:16:46 +0900
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=echo,tier=app
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=echo
           tier=app
  Containers:
   echo:
    Image:        ghcr.io/subicura/echo:v2
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  echo-deploy-5b6d56b5d4 (0/0 replicas created)
NewReplicaSet:   echo-deploy-7b47dfc99f (4/4 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  43m    deployment-controller  Scaled up replica set echo-deploy-5b6d56b5d4 to 4
  Normal  ScalingReplicaSet  9m14s  deployment-controller  Scaled up replica set echo-deploy-7b47dfc99f to 1
  Normal  ScalingReplicaSet  9m14s  deployment-controller  Scaled down replica set echo-deploy-5b6d56b5d4 to 3 from 4
  Normal  ScalingReplicaSet  9m14s  deployment-controller  Scaled up replica set echo-deploy-7b47dfc99f to 2 from 1
  Normal  ScalingReplicaSet  9m9s   deployment-controller  Scaled down replica set echo-deploy-5b6d56b5d4 to 2 from 3
  Normal  ScalingReplicaSet  9m9s   deployment-controller  Scaled up replica set echo-deploy-7b47dfc99f to 3 from 2
  Normal  ScalingReplicaSet  9m8s   deployment-controller  Scaled down replica set echo-deploy-5b6d56b5d4 to 1 from 2
  Normal  ScalingReplicaSet  9m8s   deployment-controller  Scaled up replica set echo-deploy-7b47dfc99f to 4 from 3
  Normal  ScalingReplicaSet  9m8s   deployment-controller  Scaled down replica set echo-deploy-5b6d56b5d4 to 0 from 1
```

 * Deployment Controller는 Deployment조건을 감시하면서 현재 상태와 원하는 상태가 다른 것을 체크
 * Deployment Controller가 원하는 상태가 되도록 ReplicaSet 설정
 * ReplicaSet Controller는 ReplicaSet조건을 감시하면서 현재 상태와 원하는 상태가 다른 것을 체크
 * ReplicaSet Controller가 원하는 상태가 되도록 Pod을 생성하거나 제거
 * Scheduler는 API서버를 감시하면서 할당되지 않은 Pod이 있는지 체크
 * Scheduler는 할당되지 않은 새로운 Pod을 감지하고 적절한 노드에 배치
 * 이후 노드는 기존대로 동작

Deployment는 Deployment Controller가 관리하고 ReplicaSet과 Pod은 기존 Controller와 Scheduler가 관리한다.


#### 버전관리
Deployment는 변경된 상태를 기록한다.
```shell
# 히스토리 확인
kubectl rollout history deploy/echo-deploy

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/deployment$ kubectl rollout history deploy/echo-deploy
deployment.apps/echo-deploy
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

# revision 1 히스토리 상세 확인
kubectl rollout history deploy/echo-deploy --revision=1

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/deployment$ kubectl rollout history deploy/echo-deploy --revision=1
deployment.apps/echo-deploy with revision #1
Pod Template:
  Labels:       app=echo
        pod-template-hash=5b6d56b5d4
        tier=app
  Containers:
   echo:
    Image:      ghcr.io/subicura/echo:v1
    Port:       <none>
    Host Port:  <none>
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>

# 바로 전으로 롤백
kubectl rollout undo deploy/echo-deploy

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/deployment$ kubectl rollout undo deploy/echo-deploy
deployment.apps/echo-deploy rolled back
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/deployment$ kubectl get po,rs,deploy
NAME                               READY   STATUS    RESTARTS   AGE
pod/echo-deploy-5b6d56b5d4-6tmrb   1/1     Running   0          24s
pod/echo-deploy-5b6d56b5d4-966j7   1/1     Running   0          26s
pod/echo-deploy-5b6d56b5d4-9mxzg   1/1     Running   0          24s
pod/echo-deploy-5b6d56b5d4-dkmwh   1/1     Running   0          26s

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/echo-deploy-5b6d56b5d4   4         4         4       53m
replicaset.apps/echo-deploy-7b47dfc99f   0         0         0       19m

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/echo-deploy   4/4     4            4           53m

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/deployment$ kubectl rollout history deploy/echo-deploy --revision=3
deployment.apps/echo-deploy with revision #3
Pod Template:
  Labels:       app=echo
        pod-template-hash=5b6d56b5d4
        tier=app
  Containers:
   echo:
    Image:      ghcr.io/subicura/echo:v1
    Port:       <none>
    Host Port:  <none>
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>

# 특정 버전으로 롤백
kubectl rollout undo deploy/echo-deploy --to-revision=2

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/deployment$ kubectl rollout history deploy/echo-deploy --revision=4
deployment.apps/echo-deploy with revision #4
Pod Template:
  Labels:       app=echo
        pod-template-hash=7b47dfc99f
        tier=app
  Containers:
   echo:
    Image:      ghcr.io/subicura/echo:v2 
    Port:       <none>
    Host Port:  <none>
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
```

#### 배포 전략 설정

Deployment 다양한 방식의 배포 전략이 있다. 여기선 롤링업데이트 방식을 사용할 때 동시에 업데이트하는 Pod의 개수를 변경해보자.

쿠버네티스에서는 이와 같은 무중단 배포를 지원합니다. 대표적인 3가지 방법으로 블루/그린 방법, 롤링 업데이트 방법, 카나리 배포 방법이 있다.

롤링 업데이트란 새 버전을 배포하면서, 새 버전 인스턴스를 하나씩 늘려가고 기존 버전의 인스턴스를 하나식 줄여나가는 방식이다. 
이러한 경우 새 버전의 인스턴스로 트래픽이 이전되기 전까지 이전 버전과 새 버전의 인스턴스가 동시에 존재할 수 있다는 단점이 있지만, 
시스템을 무중단으로 업데이트 할 수 있다는 장점이 있다

.spec.strategy.type에는 두 가지 가능한 필드가 존재합니다.

* RollingUpdate : 새로운 파드가 점진적으로 추가되고 이전 파드가 중지됩니다.
* Recreate : 새로운 파드가 추가되기 전에 모든 이전 파드가 한 번에 중지됩니다.


앞서 설명 들였듯 Rolling Update가 배포에 가장 적합한 업데이트 전략으로 현재 Deployment의 디폴트 배포 전략입니다. RollingUpdate에는 업데이트 프로세스를 미세하게 조정할 수 있는 두 가지 옵션이 더 존재하는데 아래와 같습니다.

* maxSurge : 업데이트 중에 동시에 최대로 추가할 수 있는 파드의 수입니다. (새로운 버전의 파드를 미리 생성해야 하므로)
* maxUnavailable : 업데이트 과정 중에서 동시에 사용 불가능한 상태가 될 수 있는 파드의 수 이다.


maxSurge와 maxUnavailable 둘 다 정수나 % 단위로 지정할 수 있지만, 둘다 0이 될 수는 없다. 
값을 정수로 지정하게 될 경우 실제로 업데이트되는 파드의 수가 지정된다. 
그리고 백분율(%)로 지정하게 되면 원하는 개수의 파드의 비율이 반올림되어 지정된다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-deploy-st
spec:
  replicas: 4
  selector:
    matchLabels:
      app: echo
      tier: app
  minReadySeconds: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 3
  template:
    metadata:
      labels:
        app: echo
        tier: app
    spec:
      containers:
        - name: echo
          image: ghcr.io/subicura/echo:v1
          livenessProbe:
            httpGet:
              path: /
              port: 3000
```

Deployment를 생성하고 결과를 확인

```shell
kubectl apply -f echo-strategy.yml

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/deployment$ kubectl apply -f echo-strategy.yml
deployment.apps/echo-deploy-st created

kubectl get po,rs,deploy

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/deployment$ kubectl get po,rs,deploy
NAME                                  READY   STATUS    RESTARTS   AGE
pod/echo-deploy-st-6d8559cdcc-plnq6   1/1     Running   0          63s
pod/echo-deploy-st-6d8559cdcc-tsb98   1/1     Running   0          63s
pod/echo-deploy-st-6d8559cdcc-xpmjh   1/1     Running   0          63s
pod/echo-deploy-st-6d8559cdcc-zxwcp   1/1     Running   0          63s

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/echo-deploy-st-6d8559cdcc   4         4         4       63s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/echo-deploy-st   4/4     4            4           63s

# 이미지 변경 (명령어로)
kubectl set image deploy/echo-deploy-st echo=ghcr.io/subicura/echo:v2

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/deployment$ kubectl set image deploy/echo-deploy-st echo=ghcr.io/subicura/echo:v2
deployment.apps/echo-deploy-st image updated

# 이벤트 확인
kubectl describe deploy/echo-deploy-st

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/deployment$ kubectl describe deploy/echo-deploy-st
Name:                   echo-deploy-st
Namespace:              default
CreationTimestamp:      Tue, 28 Nov 2023 18:43:01 +0900
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=echo,tier=app
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        5
RollingUpdateStrategy:  3 max unavailable, 3 max surge
Pod Template:
  Labels:  app=echo
           tier=app
  Containers:
   echo:
    Image:        ghcr.io/subicura/echo:v2
    Port:         <none>
    Host Port:    <none>
    Liveness:     http-get http://:3000/ delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  echo-deploy-st-6d8559cdcc (0/0 replicas created)
NewReplicaSet:   echo-deploy-st-6c7fcbbf56 (4/4 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  3m21s  deployment-controller  Scaled up replica set echo-deploy-st-6d8559cdcc to 4
  Normal  ScalingReplicaSet  76s    deployment-controller  Scaled up replica set echo-deploy-st-6c7fcbbf56 to 3 #신규로 3개의 컨테이너 기동
  Normal  ScalingReplicaSet  76s    deployment-controller  Scaled down replica set echo-deploy-st-6d8559cdcc to 1 from 4 #3개의 컨테이너 셧다운 
  Normal  ScalingReplicaSet  76s    deployment-controller  Scaled up replica set echo-deploy-st-6c7fcbbf56 to 4 from 3 #나머지 1개 컨테이너 기동
  Normal  ScalingReplicaSet  68s    deployment-controller  Scaled down replica set echo-deploy-st-6d8559cdcc to 0 from 1 #나머지 1개 컨테이너 셧다운
```
Pod을 하나씩 생성하지 않고 한번에 3개가 생성된 것을 확인할 수 있다.
maxSurge와 maxUnavailable의 기본값은 25%이다. 대부분의 상황에서 적당하지만 상황에 따라 적절하게 조정이 필요하다

#### 마무리

Deployment는 가장 흔하게 사용하는 배포방식이다. 이외에 StatefulSet, DaemonSet, CronJob, Job등이 있지만 사용법은 크게 다르지 않다.