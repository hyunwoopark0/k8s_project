# k8s_project

# EDA 
- ## data
  ```
  
  Year: The year when the flight took place.
  Month: The month when the flight took place.
  DayofMonth: The day of the month when the flight took place.
  DayOfWeek: The day of the week when the flight took place (e.g., Monday, Tuesday, etc.).
  DepTime: The actual departure time of the flight.
  CRSDepTime: The scheduled departure time of the flight.
  ArrTime: The actual arrival time of the flight.
  CRSArrTime: The scheduled arrival time of the flight.
  UniqueCarrier: A code or identifier for the airline operating the flight.
  FlightNum: The flight number of the specific flight.
  TailNum: The aircraft tail number, which uniquely identifies the aircraft.
  ActualElapsedTime: The actual time taken for the flight, including departure and arrival times.
  CRSElapsedTime: The scheduled (planned) time for the flight, including departure and arrival times.
  AirTime: The amount of time the aircraft was in the air during the flight.
  ArrDelay: The actual arrival delay in minutes.
  DepDelay: The actual departure delay in minutes.
  
  
  
  
  Origin: The code or identifier for the airport of origin.
  Dest: The code or identifier for the destination airport.
  Distance: The distance in miles between the origin and destination airports.
  TaxiIn(활주로): The time in minutes that the aircraft spent taxiing after landing.
  TaxiOut: The time in minutes that the aircraft spent taxiing before taking off.
  Cancelled: A binary indicator (0 or 1) to show if the flight was canceled.
  CancellationCode: A code representing the reason for flight cancellation (e.g., weather-related, carrier-related).
  Diverted: A binary indicator (0 or 1) to show if the flight was diverted to an alternate airport.
  CarrierDelay: The delay attributed to the airline carrier, in minutes.
  WeatherDelay: The delay attributed to weather conditions, in minutes.
  NASDelay: The delay attributed to National Airspace System (NAS) issues, in minutes.
  SecurityDelay: The delay attributed to security-related issues, in minutes.
  LateAircraftDelay: The delay attributed to a previously delayed aircraft, in minutes.
  
  ```
  
  ```
  EDA 
  
  예상 서비스
  1. 항공 정보 입력 시 도착 시간 예측
  
  2. 항공 티켓 구매 시 알 수 있는 정보들을 입력받아, 해당 비행 예상 시간 및 지연율 / 취소 확률 예측
  
  
  
  
  
  ```
  
  
  - ## 서비스
  
  ```
  'Year': 연도
  'Month': 월
  'DayofMonth': 월의 일자
  'DayOfWeek': 요일
  'CRSDepTime': 예정 출발 시간 (CRS는 Computer Reservation System의 약자로, 컴퓨터 예약 시스템에서 제공하는 스케줄 정보를 의미합니다.)
  'CRSArrTime': 예정 도착 시간
  'FlightNum': 항공편 번호
  'Distance': 거리 
  
  1. 티켓 정보 입력 시( 위 정보들은 티켓에 나와있다고 가정 ) , 예상 지연 시간 또는 지연이 될 확률 제공  ( 모델은 Random Forest 사용 -> 성능은 DE 니까 중요하지 않음.. )
  
  2. 연도 별 CSV 파일을 주기적으로 업데이트 ( 년마다 업데이트로 가정 ) , 웹 서비스에 지연율이 가장 낮은 항공사 Rank 표를 만들어서, 연도 별 항공 순위 제공
  
  ```
  - ## 예시 서비스 화면 - ( 정보 입력 시, 지연 확률 or 예상 지연 시간 제공 )
  
  
  <img width="1274" alt="스크린샷 2023-10-24 오후 1 56 42" src="https://github.com/yeardream-de-project-team4/k8s_project/assets/113021892/a6d9257d-e172-49c8-9fb2-b25a53760c24">
  
  <img width="1928" alt="스크린샷 2023-10-26 오후 12 16 07" src="https://github.com/yeardream-de-project-team4/k8s_project/assets/113021892/ce495a53-f5d5-499a-babc-ba4db614b458">







# k8s setting
- ## 모든 노드 공통

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

- ## 보안그룹 설정
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

- ## 마스터 노드

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

- ## 워커 노드
  
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
- ## build custom airflow image

  m1 Mac 의 경우 ec2와 아키텍쳐가 다르기때문에 ec2에서 빌드할 것
  
  ```
  docker login
  docker build . -t [docker hub id]/[repo]:0.3
  docker push [docker hub id]/[repo]:0.3
  ```

- ## postgres for metadata database setup

  ```
  sudo -i -u postgres
  
  psql
  
  CREATE DATABASE airflow;
  CREATE USER airflow WITH PASSWORD 'airflow';
  GRANT ALL PRIVILEGES ON DATABASE airflow TO airflow;
  \c airflow;
  GRANT ALL ON SCHEMA public TO airflow;
  
  ```

- ## pv setup

  nfs 서버의 /etc/exports 아래에 두 줄 추가 후에 nfs 서버 재시작 (디렉토리도 만들어놓아야함)
  ```
  /mnt/data/airflow/dags *(rw,sync,no_root_squash)
  /mnt/data/airflow/logs *(rw,sync,no_root_squash)
  ```

- ## airflow setup

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



  
# kafka
- ## kafka architecture in k8s
  ---
  
  <img width="1156" alt="image" src="https://github.com/yeardream-de-project-team4/k8s_project/assets/75843973/36c63c00-1cfd-4ebd-9726-2934713d4855">
  
  ---

- ## producer
  ---
  
  <img width="1209" alt="스크린샷 2023-11-02 오후 3 11 13" src="https://github.com/yeardream-de-project-team4/k8s_project/assets/75843973/c7e3a0f8-aa52-42b4-8217-43546b61d604">
  
  - producer.jar
  
  git clone --branch feature/kafka https://github.com/yeardream-de-project-team4/k8s_project.git
  
  - producer.yaml
  ```
  apiVersion: batch/v1
  kind: Job
  metadata:
    name: airport-producer-job
  spec:
    template:
      spec:
        restartPolicy: Never
        containers:
        - name: my-container
          image: 나의 producer image
          volumeMounts:
          - name: nfs-volume
            mountPath: /app/resources
        volumes:
        - name: nfs-volume
          persistentVolumeClaim:
            claimName: my-nfs-pvc
  ```
  
  ---

- ## broker
  ---
  
  <img width="946" alt="스크린샷 2023-11-02 오후 3 45 38" src="https://github.com/yeardream-de-project-team4/k8s_project/assets/75843973/2598ed83-b317-4409-9481-d8d69c7561e6">
  
  
  ※ zookeeper 및 Kafka의 config는 env 부분에 각자 원하는대로 설정하면 된다. 참고로 아래는 기본적인 설정만 되어있다.
  
  
  
  - zookeeper.yaml
  ```
  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: zookeeper-statefulset
    labels:
      app: zookeeper
  spec:
    replicas: 3
    serviceName: zksvc
    selector:
      matchLabels:
        app: zookeeper
    template:
      metadata:
        labels:
          app: zookeeper
      spec:
        containers:
        - name: zookeeper
          image: confluentinc/cp-zookeeper:7.0.1
          ports:
          - containerPort: 2181
          env:
          - name: ZOOKEEPER_CLIENT_PORT
            value: "2181"
          - name: ZOOKEEPER_TICK_TIME
            value: "2000"
    volumeClaimTemplates:
    - metadata:
        name: datadir
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "zoo-pv"
        resources:
          requests:
            storage: 10Gi
  
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: zksvc
  spec:
    type: ClusterIP
    clusterIP: None
    selector:
      app: zookeeper
    ports:
      - protocol: TCP
        port: 2181
        targetPort: 2181
  ```
  - zookeeper_pv.yaml
  ```
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: zoo-pv-0
  spec:
    capacity:
      storage: 10Gi
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Delete
    storageClassName: 'zoo-pv'
    hostPath:
      path: /data/
      type: DirectoryOrCreate
  ---
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: zoo-pv-1
  spec:
    capacity:
      storage: 10Gi
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Delete
    storageClassName: 'zoo-pv'
    hostPath:
      path: /data/
      type: DirectoryOrCreate
  ---
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: zoo-pv-2
  spec:
    capacity:
      storage: 10Gi
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Delete
    storageClassName: 'zoo-pv'
    hostPath:
      path: /data/
      type: DirectoryOrCreate
  ```
  - Kafka.yaml
  ```
  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: kafka-statefulset
    labels:
      app: kafka
  spec:
    replicas: 3
    serviceName: kafka-service
    selector:
      matchLabels:
        app: kafka
    template:
      metadata:
        labels:
          app: kafka
      spec:
        containers:
        - name: broker
          image: confluentinc/cp-kafka:7.0.1
          ports:
          - containerPort: 9092
          env:
          - name: MY_POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: KAFKA_ZOOKEEPER_CONNECT
            value: zookeeper-statefulset-0.zksvc:2181,zookeeper-statefulset-1.zksvc:2181,zookeeper-statefulset-2.zksvc:2181
          - name: KAFKA_LISTENER_SECURITY_PROTOCOL_MAP
            value: PLAINTEXT:PLAINTEXT,PLAINTEXT_INTERNAL:PLAINTEXT
          - name: KAFKA_ADVERTISED_LISTENERS
            value: PLAINTEXT://:29092,PLAINTEXT_INTERNAL://$(MY_POD_NAME).kafka-service:9092
          - name: KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR
            value: "1"
          - name: KAFKA_TRANSACTION_STATE_LOG_MIN_ISR
            value: "1"
          - name: KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR
            value: "1"
    volumeClaimTemplates:
    - metadata:
        name: kafkadir
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: "kafka"
        resources:
          requests:
            storage: 10Gi
  
  ---
  
  apiVersion: v1
  kind: Service
  metadata:
    name: kafka-service
  spec:
    type: ClusterIP
    clusterIP: None
    selector:
      app: kafka
    ports:
      - protocol: TCP
        port: 9092
        targetPort: 9092
  ```
  - Kafka_pv.yaml
  ```
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: kafka-0
    namespace: kafka-ns
  spec:
    capacity:
      storage: 10Gi
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Delete
    storageClassName: "kafka"
    hostPath:
      path: /data/
      type: DirectoryOrCreate
  ---
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: kafka-1
    namespace: kafka-ns
  spec:
    capacity:
      storage: 10Gi
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Delete
    storageClassName: "kafka"
    hostPath:
      path: /data/
      type: DirectoryOrCreate
  ---
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: kafka-2
    namespace: kafka-ns
  spec:
    capacity:
      storage: 10Gi
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Delete
    storageClassName: "kafka"
    hostPath:
      path: /data/
      type: DirectoryOrCreate
  ```
  
  
  ---


- ## consumer
  ---
  
  <img width="977" alt="스크린샷 2023-11-02 오후 2 19 28" src="https://github.com/yeardream-de-project-team4/k8s_project/assets/75843973/5371e178-bcc2-4627-99a4-2bf043c92c0b">
  
  - consumer.jar
  
  git clone --branch feature/kafka https://github.com/yeardream-de-project-team4/k8s_project.git
  
  - consumer.yaml
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: airport-consumer
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: my-app
    template:
      metadata:
        labels:
          app: my-app
      spec:
        containers:
          - name: airport-consumer
            image: 나의 consumer image
            ports:
              - containerPort: 80
  ```
  
  ---



