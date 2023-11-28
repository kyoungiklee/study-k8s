
# 준비

## 윈도우10에 wsl2 설치 및 Unbuntu 22.04 설치
참조 https://llighter.github.io/install_wsl2/

간략 절차
* Windows PowerShell 관리자 권한으로 실행
* Linux용 Windows 하위 시스템 옵션 기능을 사용하도록 설정
    * dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
* Virtual Machine 기능 사용
    * dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
* Linux 커널 업데이트 패키지 다운로드
    *  https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi
* WSL 2를 기본 버전으로 설정
    * wsl --set-default-version 2
* . Microsoft Store에서 Unbuntu 22.04 설치


## 실습을 위한 쿠버네티스 클러스터와 kubectl 설치
### 쿠버네티스 설치

#### minikube
쿠버네티스를 운영환경에 설치하기 위해서는 최소 3대의 마스터 서버와 컨테이너 배포를 위한 n개의 노드 서버가 필요하다.

개발환경에서는 마스터와 노드를 하나의 서버에 설치하여 손쉽게 관리할 수 있도록 구성한다.

쿠버네티스 클러스터를 실행하려면 최소한 scheduler, controller, api-server, etcd, kubelet, kube-proxy를 설치해야 하고
필요에 따라 dns, ingress controller, storage class등을 설치해야

이러한 설치를 쉽고 빠르게 하기 위한 도구가 minikube 이다. (그외 k3s, docker for desktop, kind 등이 있다.)

```shell
# docker 사용시 설치 필요, docker를 사용하지 않는 경우 virtual box 설치
curl -fsSL https://get.docker.com/ | sudo sh
sudo usermod -aG docker $USER

# docker 대신 virtual box 설치
sudo apt-get install virtualbox

# install minikube
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin
```
기본 명령어

```shell
# 버전확인
minikube version

# 가상머신 시작
minikube start --driver=docker
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide$ minikube start --driver=docker
😄  minikube v1.32.0 on Ubuntu 22.04 (amd64)
✨  Using the docker driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔄  Restarting existing docker container for "minikube" ...
🐳  Preparing Kubernetes v1.28.3 on Docker 24.0.7 ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
💡  kubectl not found. If you need it, try: 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default


# driver 에러가 발생한다면 virtual box를 사용
minikube start --driver=virtualbox
# 특정 k8s 버전 실행
minikube start --kubernetes-version=v1.23.1

# 상태확인
minikube status

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

# 정지
minikube stop

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide$ minikube stop
✋  Stopping node "minikube"  ...
🛑  Powering off "minikube" via SSH ...
🛑  1 node stopped.

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide$ minikube status
minikube
type: Control Plane
host: Stopped
kubelet: Stopped
apiserver: Stopped
kubeconfig: Stopped
# 삭제
minikube delete

# ssh 접속
minikube ssh
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide$ minikube ssh
docker@minikube:~$

# ip 확인
minikube ip
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide$ minikube ip
192.168.49.2
```

#### kubectl 설치
kubectl 은 쿠버네티스 CLI 도구이다. 

리눅스 설치
```shell
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```
테스트
```shell
kubectl version

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide$ kubectl version
Client Version: v1.28.3
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.28.3
```

#### 유용한 도구들
* [kubectx](https://github.com/ahmetb/kubectx) - 컨텍스트 전환 CLI
* [kubens](https://github.com/ahmetb/kubectx) - namespace 전환 CLI
* [k9s](https://github.com/derailed/k9s)- 클러스터 관리 CLI
* [kubespy](https://github.com/pulumi/kubespy)- 쿠버네티스 상태를 실시간으로 확인합니다.
* [stern](https://github.com/wercker/stern)- 통합 로그 관리 도구
* [Lens](https://k8slens.dev/)- 클러스터 관리 GUI

#### docker vs k8s 배포
docker 에 배포
```yaml
version: "3"

services:
  wordpress:
    image: wordpress:5.9.1-php8.1-apache
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: password
    ports:
      - "30000:80"

  mysql:
    image: mariadb:10.7
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_ROOT_PASSWORD: password
```

k8s 에 배포
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
        - image: mariadb:10.7
          name: mysql
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
        - image: wordpress:5.9.1-php8.1-apache
          name: wordpress
          env:
            - name: WORDPRESS_DB_HOST
              value: wordpress-mysql
            - name: WORDPRESS_DB_NAME
              value: wordpress
            - name: WORDPRESS_DB_USER
              value: root
            - name: WORDPRESS_DB_PASSWORD
              value: password
          ports:
            - containerPort: 80
              name: wordpress

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
위 소스를 wordpress-k8s.yml 파일로 저장하고 다음과 같은 명령어를 실행한다.
```shell
# wordpress-k8s.yml 설정 적용
kubectl apply -f wordpress-k8s.yml

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl apply -f wordpress-k8s.yml
deployment.apps/wordpress-mysql created
service/wordpress-mysql created
deployment.apps/wordpress created
service/wordpress created
```

배포상태를 확인한다.
```shell
# 현재 상태 확인
kubectl get all

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl get all
NAME                                   READY   STATUS    RESTARTS   AGE
pod/wordpress-746bd6d54b-5ncm7         1/1     Running   0          34s
pod/wordpress-mysql-78488dd7d5-q27b8   1/1     Running   0          34s

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP        18h
service/wordpress         NodePort    10.109.36.5      <none>        80:30357/TCP   34s
service/wordpress-mysql   ClusterIP   10.102.210.205   <none>        3306/TCP       34s

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/wordpress         1/1     1            1           34s
deployment.apps/wordpress-mysql   1/1     1            1           34s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/wordpress-746bd6d54b         1         1         1       34s
replicaset.apps/wordpress-mysql-78488dd7d5   1         1         1       34s
```
Docker driver를 사용중이라면 minikube service wordpress 명령어를 이용하여 접속한다.

```shell
roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ minikube service wordpress
|-----------|-----------|-------------|---------------------------|
| NAMESPACE |   NAME    | TARGET PORT |            URL            |
|-----------|-----------|-------------|---------------------------|
| default   | wordpress |          80 | http://192.168.49.2:30357 |
|-----------|-----------|-------------|---------------------------|
🏃  Starting tunnel for service wordpress.
|-----------|-----------|-------------|------------------------|
| NAMESPACE |   NAME    | TARGET PORT |          URL           |
|-----------|-----------|-------------|------------------------|
| default   | wordpress |             | http://127.0.0.1:35739 |
|-----------|-----------|-------------|------------------------|
🎉  Opening service default/wordpress in default browser...
👉  http://127.0.0.1:35739
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.
```
#### 마무리

```shell
kubectl delete -f wordpress-k8s.yml

roadseeker@DESKTOP-ARE0NV9:~/k8s_guide/index$ kubectl delete -f wordpress-k8s.yml
deployment.apps "wordpress-mysql" deleted
service "wordpress-mysql" deleted
deployment.apps "wordpress" deleted
service "wordpress" deleted
```














