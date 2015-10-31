# Docker & k8s meetup resource repository


### etcd-init scripts

```

#!/usr/bin/bash

if [ $# -ne 3 ]; then
	echo 'HELP: '
	echo './init-etcd ETCD_NAME HOST_IP CLUSTER_INFO'
	exit 0;
fi
#ETCD_NAME=etcd4
ETCD_NAME=$1
#HOST_IP=10.12.1.104
HOST_IP=$2
#CLUSTER=etcd0=http://10.12.1.100:2380,etcd1=http://10.12.1.101:2380,etcd2=http://10.12.1.102:2380,etcd3=http://10.12.1.103:2380,etcd4=http://10.12.1.104:2380
CLUSTER=$3
TOKEN=etcd-cluster-1

#init etcd2 data folder
DATA_DIR=/var/lib/etcd2
sudo mkdir -p ${DATA_DIR}
sudo chown -Rh etcd:etcd ${DATA_DIR}


cat <<EOF > /tmp/cloud-init-etcd-example.yaml
==== CoreOS cloud init script START ====
coreos:
  etcd2:
    name: ${ETCD_NAME}
    advertise-client-urls: http://${HOST_IP}:2379
    listen-peer-urls: http://${HOST_IP}:2380
    listen-client-urls: http://${HOST_IP}:2379,http://127.0.0.1:2379
    initial-advertise-peer-urls: http://${HOST_IP}:2380
    initial-cluster: ${CLUSTER}
    data-dir: ${DATA_DIR}
  units:
    - name: etcd2.service
      command: start
      
==== CoreOS cloud init script END ====

EOF

cat /tmp/cloud-init-etcd-example.yaml

function pause() {
    prompt="$1"
    echo -e -n "\033[1;36m$prompt"
    echo -e -n '\033[0m'
    read
    clear
}

pause "Press enter to continue...\n"


sudo -u etcd /bin/etcd2 \
  -name ${ETCD_NAME} \
  -advertise-client-urls http://${HOST_IP}:2379 \
  -listen-peer-urls http://${HOST_IP}:2380 \
  -listen-client-urls http://${HOST_IP}:2379,http://127.0.0.1:2379 \
  -initial-advertise-peer-urls http://${HOST_IP}:2380 \
  -initial-cluster-token ${TOKEN} \
  -initial-cluster ${CLUSTER} \
  -initial-cluster-state new \
  -data-dir ${DATA_DIR}

```



### etcd-proxy mode example

```
mkdir -p /var/lib/etcd2
chown -Rh etcd:etcd /var/lib/etcd2
sudo -u etcd /bin/etcd2 -proxy on \
-listen-client-urls http://127.0.0.1:4001 \
-initial-cluster etcd0=http://10.12.1.100:2380,etcd1=http://10.12.1.101:2380,etcd2=http://10.12.1.102:2380,etcd3=http://10.12.1.103:2380,etcd4=http://10.12.1.104:2380 \
-data-dir /var/lib/etcd2

#cloud init example

  etcd2:
    proxy: on
    listen-client-urls: http://127.0.0.1:4001
    initial-cluster: etcd0=http://10.12.1.100:2380,etcd1=http://10.12.1.101:2380,etcd2=http://10.12.1.102:2380,etcd3=http://10.12.1.103:2380,etcd4=http://10.12.1.104:2380
    data-dir: /var/lib/etcd2

```


