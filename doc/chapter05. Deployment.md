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
