
## 준비
실습을 위한 쿠버네티스 클러스터와 kubectl 설치
* 쿠버네티스 설치

#### minikube
쿠버네티스를 운영환경에 설치하기 위해서는 최소 3대의 마스터 서버와 컨테이너 배포를 위한 n개의 노드 서버가 필요하다.

개발환경에서는 마스터와 노드를 하나의 서버에 설치하여 손쉽게 관리할 수 있도록 구성한다.

쿠버네티스 클러스터를 실행하려면 최소한 scheduler, controller, api-server, etcd, kubelet, kube-proxy를 설치해야 하고
필요에 따라 dns, ingress controller, storage class등을 설치해야

이러한 설치를 쉽고 빠르게 하기 위한 도구가 minikube 이다. (그외 k3s, docker for desktop, kind 등이 있다.)

#### 윈도우10에 wsl2 설치 및 Unbuntu 22.04 설치
참조 https://llighter.github.io/install_wsl2/

간략 절차
* Windows PowerShell 관리자 권한으로 실행
* Linux용 Windows 하위 시스템 옵션 기능을 사용하도록 설정 - dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
* 