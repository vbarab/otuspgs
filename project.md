Собираем гластер, повторяем на всех нодах
```
udo apt update
sudo apt-get dist-upgrade
`sudo apt install lvm2`

mkdir -p /data/brick1

mkdir -p /data/brick1/gv1

mkdir -p /data/brick2/gv2

`sudo pvcreate /dev/sdb`

vgcreate vgroup1 /dev/sdb

lvcreate -L 15G -n lv01 vgroup1

lvcreate -L 15G -n lv02 vgroup1

mkfs.xfs -i size=512 /dev/vgroup1/lv01

mkfs.xfs -i size=512 /dev/vgroup1/lv02

sudo blkid

nano /etc/fstab

UID=7d77fbb9-a3b5-46f6-a91f-028ef60809a0       /data/brick1   xfs   defaults   1 2
UUID=7fef7897-9174-4dfc-b7f0-4df6dbd2ec18       /data/brick2    xfs     defaults   1 2

mount -a

#Если не маунтится то

systemctl daemon-reload
```
Далее на любой ноде (я бахал с первой)

```
sudo gluster peer probe gluster2
sudo gluster peer probe gluster3
sudo gluster peer probe gluster4

gluster volume create gv1 replica 4 gluster1:/data/brick1/gv1 gluster2:/data/brick1/gv1 gluster3:/data/brick1/gv1 gluster4:/data/brick1/gv1 force
gluster volume create gv2 replica 4 gluster1:/data/brick2/gv2 gluster2:/data/brick2/gv2 gluster3:/data/brick2/gv2 gluster4:/data/brick2/gv2
gluster volume start gv1 && gluster volume start gv2
```
На клиентах где будем юзать гластер
```

sudo apt install glusterfs-client -y 
mkdir -p /mnt/gluster/patroni
mkdir -p /mnt/gluster/postgres


gluster1:/gv1 /mnt/gluster/patroni glusterfs defaults,_netdev,backupvolfile-server=gluster2:gluster3:gluster4 0 0
gluster1:/gv2 /mnt/gluster/postgres glusterfs defaults,_netdev,backupvolfile-server=gluster2:gluster3:gluster4 0 0
```
Поднимам etcd
```
sudo apt-get install etcd -y

sudo mkdir /opt/etcd


#Демо вариант, для норм настрйоки нужно сконфигурить файлы сервиса

TOKEN=token-01
CLUSTER_STATE=new
NAME_1=etcd1
NAME_2=etcd2
NAME_3=etcd3
HOST_1=172.27.65.58
HOST_2=172.27.65.55
HOST_3=172.27.65.66
ENDPOINTS=$HOST_1:2379,$HOST_2:2379,$HOST_3:2379
CLUSTER=${NAME_1}=http://${HOST_1}:2380,${NAME_2}=http://${HOST_2}:2380,${NAME_3}=http://${HOST_3}:2380

# For machine 1
THIS_NAME=${NAME_1}
THIS_IP=${HOST_1}
etcd --data-dir="/opt/etcd" --name ${THIS_NAME} \
	--initial-advertise-peer-urls http://${THIS_IP}:2380 --listen-peer-urls http://${THIS_IP}:2380 \
	--advertise-client-urls http://${THIS_IP}:2379 --listen-client-urls http://${THIS_IP}:2379 \
	--initial-cluster ${CLUSTER} \
	--initial-cluster-state ${CLUSTER_STATE} --initial-cluster-token ${TOKEN}

# For machine 2
THIS_NAME=${NAME_2}
THIS_IP=${HOST_2}
etcd --data-dir="/opt/etcd" --name ${THIS_NAME} \
	--initial-advertise-peer-urls http://${THIS_IP}:2380 --listen-peer-urls http://${THIS_IP}:2380 \
	--advertise-client-urls http://${THIS_IP}:2379 --listen-client-urls http://${THIS_IP}:2379 \
	--initial-cluster ${CLUSTER} \
	--initial-cluster-state ${CLUSTER_STATE} --initial-cluster-token ${TOKEN}

# For machine 3
THIS_NAME=${NAME_3}
THIS_IP=${HOST_3}
etcd --data-dir="/opt/etcd" --name ${THIS_NAME} \
	--initial-advertise-peer-urls http://${THIS_IP}:2380 --listen-peer-urls http://${THIS_IP}:2380 \
	--advertise-client-urls http://${THIS_IP}:2379 --listen-client-urls http://${THIS_IP}:2379 \
	--initial-cluster ${CLUSTER} \
	--initial-cluster-state ${CLUSTER_STATE} --initial-cluster-token ${TOKEN}

# а потом конфигурим уже по образу и подобию все 3 ноды

ETCD_LISTEN_PEER_URLS="http://172.27.65.58:2380"
ETCD_LISTEN_CLIENT_URLS="http://localhost:2379,http://172.27.65.58:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://172.27.65.58:2380"
ETCD_INITIAL_CLUSTER="etcd1=http://172.27.65.58:2380,etcd2=http://172.27.65.55:2380,etcd3=http://172.27.65.66:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://172.27.65.58:2379"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_DATA_DIR="/opt/etcd/"
ETCD_ENABLE_V2="true"
```
Команды для проверки состояния
```
ENDPOINTS=$HOST_1:2379,$HOST_2:2379,$HOST_3:2379

etcdctl --endpoints=$ENDPOINTS member list
etcdctl --endpoints=$ENDPOINTS endpoint health
etcdctl --endpoints=$ENDPOINTS endpoint list
```
Если возникают траблы - дропаем кластер, дропаем базу данных пересоздаем кластер и добавляем ноды по 1
```
ETCD_INITIAL_CLUSTER_STATE="existing"
```
Ставим патрони
```
sudo apt-get install patroni -y
sudo apt-get install postgresql-14 -y

sudo systemctl stop postgresql
sudo systemctl disable postgresql

sudo apt-get install python-etcd -y

```

Далее конфигурим файлы на каждой ноде по образу и подобию
```
scope: postgres
namespace: /cluster/
name: patroni1

restapi:
    listen: 172.27.65.15:8008
    connect_address: 172.27.65.15:8008

etcd:
    hosts: 172.27.65.58:2379,172.27.65.55:2379,172.27.65.66:2379

bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576

  initdb:
  - encoding: UTF8
  - data-checksums

  pg_hba:
  - host replication replicator 127.0.0.1/32 md5
  - host replication replicator 172.27.65.15/0 md5
  - host replication replicator 172.27.65.21/0 md5
  - host replication replicator 172.27.65.66/0 md5
  - host all all 0.0.0.0/0 md5

  users:
    admin:
      password: admin
      options:
        - createrole
        - createdb

postgresql:
  listen: 172.27.65.15:5432
  connect_address: 172.27.65.15:5432
  data_dir: /data/patroni
  pgpass: /tmp/pgpass
  bin_dir:  /usr/lib/postgresql/14/bin/
  authentication:
    replication:
      username: replicator
      password: replicator
    superuser:
      username: postgres
      password: postgres

  parameters:
      unix_socket_directories: '.'

tags:
    nofailover: false
    noloadbalance: false
    clonefrom: false
    nosync: false

```
попаболь - проблемы с кластером были до тех пор пока не была добавлена строка
```
  bin_dir:  /usr/lib/postgresql/14/bin/
```

