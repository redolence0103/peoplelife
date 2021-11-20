# cluster backup and restore
### Controller Node 확인
```bash
kubectl get nodes
NAME                 STATUS   ROLES                  AGE     VERSION
kind-control-plane   Ready    control-plane,master   2d10h   v1.21.1
kind-worker          Ready    <none>                 2d10h   v1.21.1
kind-worker2         Ready    <none>                 2d10h   v1.21.1
kind-worker3         Ready    <none>                 2d10h   v1.21.1

```
### Controller Node로 login
```bash
docker exec -it kind-control-plane /bin/bash
which etcdctl
#없을 경우 설치
sudo apt update
sudo apt install -y etcd-client
```
### etcd process 확인
```bash
ps -ef |grep -i etcd
root         436     335  3 12:37 ?        00:03:43 etcd --advertise-client-urls=https://172.18.0.2:2379 \
--cert-file=/etc/kubernetes/pki/etcd/server.crt \
--client-cert-auth=true --data-dir=/var/lib/etcd --initial-advertise-peer-urls=https://172.18.0.2:2380 \
--initial-cluster=kind-control-plane=https://172.18.0.2:2380 --key-file=/etc/kubernetes/pki/etcd/server.key \
--listen-client-urls=https://127.0.0.1:2379,https://172.18.0.2:2379 --listen-metrics-urls=http://127.0.0.1:2381 \
--listen-peer-urls=https://172.18.0.2:2380 --name=kind-control-plane --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt \
--peer-client-cert-auth=true --peer-key-file=/etc/kubernetes/pki/etcd/peer.key --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt \
--snapshot-count=10000 --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt

```
주) --cert-file, --key-file, --trusted-ca-file 값 확인
### etcd backup (Test를 위해 namespace와 pod를 생성 한후 백업을 진행한다.)
Hint) ETCDCTL_API=3 etcdctl snapshot backup --help를 이용하여 사용 옵션 점검
```
ETCDCTL_API=3 etcdctl snapshot save /var/lib/etcd-backup01 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt \
--key==/etc/kubernetes/pki/etcd/server.key 

2021-11-20 14:46:49.180805 I | clientv3: opened snapshot stream; downloading
2021-11-20 14:46:49.328951 I | clientv3: completed snapshot read; closing
Snapshot saved at /var/lib/etcd-backup01
```
### etcd restore (Test로 생성한 namespace와 pod를 삭제후 리스토어를 진행한다.)
Hint) /etc/kubernetes/manifests/etcd.yaml 파일을 확인 하거나 etcd process 확인
```bash
ETCDCTL_API=3 etcdctl snapshot restore /var/lib/etcd-backup01 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key --data-dir=/opt/etcd-backup01 --initial-advertise-peer-urls=https://172.18.0.2:2380 \
--initial-cluster=kind-control-plane=https://172.18.0.2:2380 --name=kind-control-plane
2021-11-20 15:02:59.467576 I | mvcc: restore compact to 76868
2021-11-20 15:02:59.482032 I | etcdserver/membership: added member e58c878e0e01014 [https://172.18.0.2:2380] to cluster c74448475845f0fb
```
### etcd.yaml 파일을 편집하여 restore directory로 연결
```bash
vi /etc/kubernetes/manifests/etcd.yaml
....
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /opt/etcd-backup01
      type: DirectoryOrCreate
    name: etcd-data
....
:wq!

```
### 삭제된 namespace와 pod가 새로 생성되었는지 확인한다.
