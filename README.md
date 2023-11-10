# k8s_project

# k8s
## 모든 노드 공통

```
# 모든 노드 동일한 ssh public key 사용
ssh-keygen -t rsa
cat >> ~/.ssh/authorized_keys < ~/.ssh/id_rsa.pub

# docker 설치
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo usermod -aG docker $USER

# 브릿지 네트워크 인터페이스에 대한 트래픽이 iptables 규칙에 의해 처리되도록함
sudo modprobe br_netfilter
sudo sysctl net.bridge.bridge-nf-call-iptables=1

# 커널이 처리하는 패킷을 외부로 포워딩(IP forwarding)가능
sudo sysctl net.ipv4.ip_forward=1

sudo vim /etc/sysctl.conf
# 아래 두 줄 추가
# net.bridge.bridge-nf-call-iptables = 1
# net.ipv4.ip_forward = 1

sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null

sudo vim /etc/containerd/config.toml
# SystemdCgroup = true 설정

sudo systemctl restart containerd
sudo systemctl enable containerd

# active (running) 체크
sudo systemctl status containerd

# 쿠버네티스 설치
sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl

sudo mkdir -p /etc/apt/keyrings

echo "deb [signed-by=/etc/apt/keyrings/kubernetes.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes.gpg

sudo apt-get update
sudo apt-get install -y kubelet=1.25.6-00 kubeadm=1.25.6-00 kubectl=1.25.6-00
sudo apt-mark hold kubelet kubeadm kubectl

# 설치확인
sudo -i
kubelet --version
kubeadm version
kubectl version --output=yaml

# 여기서 AMI 로 스냅샷
```

## 보안그룹 설정
인바운드 규칙

```
유형: SSH
소스: 내 IP
설명: 인스턴스에 접속하기 위해 22번 포트 개방
```

```
유형: 모든 트래픽
소스: 현재 보안그룹
설명: 인스턴스간 통신을 위해 보안그룹내 모든 트래픽 개방
```

## 마스터 노드

```
# root 계정으로 진행, 현재 root 계정이 아니라면 sudo -i 로 root 계정으로 진입

# 호스트이름 설정
sudo hostnamectl set-hostname [마스터 노드 이름]

# 마스터 노드 설정
kubeadm config images pull --cri-socket /run/containerd/containerd.sock

# 클러스터 시작 - [중요!] 실행 후 나오는 클러스터 join 메세지 따로 저장 (kubeadm join ~~)
kubeadm init --apiserver-advertise-address=[마스터노드 private ip] --pod-network-cidr=192.168.0.0/16 --cri-socket /run/containerd/containerd.sock

# 에러발생시 아래 두 명령어 실행 후 다시 시도
# sudo modprobe br_netfilter
# sudo sysctl net.bridge.bridge-nf-call-iptables=1

# root 계정에서 나옴
exit

# config 설정
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# calico 설치
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml -O
kubectl create -f custom-resources.yaml

# 모든 pod 의 STATUS가 running 이 될때까지 대기. 이후 control + c 로 exit
watch kubectl get pods -n calico-system

# 노드 확인
kubectl get node

# [옵션] 마스터 노드에도 pod 를 띄울 수 있게 설정
kubectl taint nodes --all node-role.kubernetes.io/control-plane-

# <none > 이 출력되어야함
kubectl describe node [마스터 노드 이름] | grep Taints

# helm 설치
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

## 워커 노드

```
# 호스트이름 설정
sudo hostnamectl set-hostname [워커 노드 이름]

# 쿠버네티스 config 설정
mkdir -p $HOME/.kube
scp -p 마스터노드유저@마스터노드ip:~/.kube/config ~/.kube/config

# root 계정 진입
sudo -i

# 따로 저장해두었던 kubeadm join ~ 명령어 실행
kubeadm join ~~~

# 에러발생시 아래 두 명령어 실행 후 다시 시도
# sudo modprobe br_netfilter
# sudo sysctl net.bridge.bridge-nf-call-iptables=1

# 마스터 노드에서 kubectl get node 로 노드 추가 확인
```


# airflow
---

## build custom airflow image

m1 Mac 의 경우 ec2와 아키텍쳐가 다르기때문에 ec2에서 빌드할 것

```
docker login
docker build . -t [docker hub id]/[repo]:0.3
docker push [docker hub id]/[repo]:0.3
```

## postgres for metadata database setup

```
sudo -i -u postgres

psql

CREATE DATABASE airflow;
CREATE USER airflow WITH PASSWORD 'airflow';
GRANT ALL PRIVILEGES ON DATABASE airflow TO airflow;
\c airflow;
GRANT ALL ON SCHEMA public TO airflow;

```

## pv setup

nfs 서버의 /etc/exports 아래에 두 줄 추가 후에 nfs 서버 재시작 (디렉토리도 만들어놓아야함)
```
/mnt/data/airflow/dags *(rw,sync,no_root_squash)
/mnt/data/airflow/logs *(rw,sync,no_root_squash)
```

## airflow setup

```
# Makefile 로 설치
make

# uninstall
make clean
----------------------------------------------------------------
# Manual Install
# repo 추가
helm repo add apache-airflow https://airflow.apache.org

# pv 추가 (nfs 서버 ip 확인)
kubectl apply -f pv-airflow-dags.yaml

# [values.yaml 파일 수정]
# 첫 실행시는 아래를 true(기본값이 true), 그 이후에는 false
# migrateDatabaseJob:
#   enabled: true
#
# data: 아래에 외부 postgres db 설정
helm upgrade --install airflow apache-airflow/airflow --namespace airflow --create-namespace -f values.yaml

# uninstall
helm uninstall airflow -n airflow
kubectl delete pv pv-airflow-dags.yaml
```


