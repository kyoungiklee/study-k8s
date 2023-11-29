# 실습

### 워드프레스 배포

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
        - name: wordpress
          image: wordpress:5.9.1-php8.1-apache
          env:
            - name: WORDPRESS_DB_HOST
              value: wordpress-mysql
            - name: WORDPRESS_DB_NAME
              value: wordpress
            - name: WORDPRESS_DB_USER
              value: root
            - name: WORDPRESS_DB_PASSWORD
              value: password

---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  type: NodePort
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
```

```shell
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/example$ kubectl apply -f wordpress.yml
deployment.apps/wordpress-mysql created
service/wordpress-mysql created
deployment.apps/wordpress created
service/wordpress created
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/example$ kubectl get all
NAME                                   READY   STATUS    RESTARTS   AGE
pod/wordpress-56bddcc46b-rvhfn         1/1     Running   0          7s
pod/wordpress-mysql-66f5df59bc-qb755   1/1     Running   0          7s

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP        2d
service/wordpress         NodePort    10.102.223.222   <none>        80:32467/TCP   7s
service/wordpress-mysql   ClusterIP   10.105.89.219    <none>        3306/TCP       7s

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/wordpress         1/1     1            1           7s
deployment.apps/wordpress-mysql   1/1     1            1           7s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/wordpress-56bddcc46b         1         1         1       7s
replicaset.apps/wordpress-mysql-66f5df59bc   1         1         1       7s
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/example$ minikube service wordpress
|-----------|-----------|-------------|---------------------------|
| NAMESPACE |   NAME    | TARGET PORT |            URL            |
|-----------|-----------|-------------|---------------------------|
| default   | wordpress |          80 | http://192.168.49.2:32467 |
|-----------|-----------|-------------|---------------------------|
🏃  Starting tunnel for service wordpress.
|-----------|-----------|-------------|------------------------|
| NAMESPACE |   NAME    | TARGET PORT |          URL           |
|-----------|-----------|-------------|------------------------|
| default   | wordpress |             | http://127.0.0.1:41975 |
|-----------|-----------|-------------|------------------------|
🎉  Opening service default/wordpress in default browser...
👉  http://127.0.0.1:41975
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.


roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/example$ kubectl delete -f wordpress.yml
deployment.apps "wordpress-mysql" deleted
service "wordpress-mysql" deleted
deployment.apps "wordpress" deleted
service "wordpress" deleted
```

####  투표 애플리케이션 배포
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  template:
    metadata:
      name: mysql
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
        - name: mysql
          image: mariadb:10.7
          env:
            - name: MYSQL_DATABASE
              value: wordpress
            - name: MYSQL_ROOT_PASSWORD
              value: password
          ports:
            - containerPort: 3306
              name: mysql

---
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
        - name: wordpress
          image: wordpress:5.9.1-php8.1-apache
          env:
            - name: WORDPRESS_DB_HOST
              value: wordpress-mysql
            - name: WORDPRESS_DB_NAME
              value: wordpress
            - name: WORDPRESS_DB_USER
              value: root
            - name: WORDPRESS_DB_PASSWORD
              value: password

---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  type: NodePort
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
```

```shell

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/example$ kubectl apply -f vote.yml
deployment.apps/redis created
service/redis created
deployment.apps/postgres created
service/postgres created
deployment.apps/worker created
deployment.apps/vote created
service/vote created
deployment.apps/result created
service/result created

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/example$ kubectl get all
NAME                            READY   STATUS    RESTARTS   AGE
pod/postgres-5d9856d97c-wb9dw   1/1     Running   0          21s
pod/redis-d986f6564-qmjlk       1/1     Running   0          21s
pod/result-6bcbd88878-9klnr     1/1     Running   0          21s
pod/vote-7f4fcdf4b4-zpldk       1/1     Running   0          21s
pod/worker-574b8d756f-bl59w     1/1     Running   0          21s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        2d
service/postgres     ClusterIP   10.111.106.117   <none>        5432/TCP       21s
service/redis        ClusterIP   10.105.8.40      <none>        6379/TCP       21s
service/result       NodePort    10.108.166.154   <none>        80:31001/TCP   21s
service/vote         NodePort    10.96.66.128     <none>        80:31000/TCP   21s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/postgres   1/1     1            1           21s
deployment.apps/redis      1/1     1            1           21s
deployment.apps/result     1/1     1            1           21s
deployment.apps/vote       1/1     1            1           21s
deployment.apps/worker     1/1     1            1           21s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/postgres-5d9856d97c   1         1         1       21s
replicaset.apps/redis-d986f6564       1         1         1       21s
replicaset.apps/result-6bcbd88878     1         1         1       21s
replicaset.apps/vote-7f4fcdf4b4       1         1         1       21s
replicaset.apps/worker-574b8d756f     1         1         1       21s


roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/example$ minikube service vote &
[3] 2595842
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/example$ |-----------|------|-------------|---------------------------|
| NAMESPACE | NAME | TARGET PORT |            URL            |
|-----------|------|-------------|---------------------------|
| default   | vote |          80 | http://192.168.49.2:31000 |
|-----------|------|-------------|---------------------------|
🏃  Starting tunnel for service vote.
|-----------|------|-------------|------------------------|
| NAMESPACE | NAME | TARGET PORT |          URL           |
|-----------|------|-------------|------------------------|
| default   | vote |             | http://127.0.0.1:43521 |
|-----------|------|-------------|------------------------|
🎉  Opening service default/vote in default browser...
👉  http://127.0.0.1:43521
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.


roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/example$ minikube service result &
[4] 2596509
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/example$ |-----------|--------|-------------|---------------------------|
| NAMESPACE |  NAME  | TARGET PORT |            URL            |
|-----------|--------|-------------|---------------------------|
| default   | result |          80 | http://192.168.49.2:31001 |
|-----------|--------|-------------|---------------------------|
🏃  Starting tunnel for service result.
|-----------|--------|-------------|------------------------|
| NAMESPACE |  NAME  | TARGET PORT |          URL           |
|-----------|--------|-------------|------------------------|
| default   | result |             | http://127.0.0.1:46765 |
|-----------|--------|-------------|------------------------|
🎉  Opening service default/result in default browser...
👉  http://127.0.0.1:46765
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.

```