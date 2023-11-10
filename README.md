# k8s_project


airflow
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

---
