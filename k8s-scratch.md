# 쿠버네티스 내부구조

## Docker 설치
```bash
sudo -i
swapoff -a
sysctl -w net.ipv4.ip_forward=1
cat /proc/sys/net/ipv4/ip_forward
apt install docker.io -y
```
## K8S component 다운로드
```bash
wget https://storage.googleapis.com/kubernetes-release/release/v1.9.11/kubernetes-server-linux-amd64.tar.gz
tar -xzf kubernetes-server-linux-amd64.tar.gz
cd kubernetes/server/bin/
mv kubectl kubelet kube-apiserver kube-controller-manager kube-scheduler kube-proxy /usr/bin/
```

## kubelet component 실행
```bash
kubectl
rm *.gz
mkdir -p /etc/kubernetes/manifests
kubelet --pod-manifest-path /etc/kubernetes/manifests &> /etc/kubernetes/kubelet.log &
ps -au | grep kubelet
```
### kubelet component 동작 확인
```bash
head /etc/kubernetes/kubelet.log
cat <<EOF > /etc/kubernetes/manifests/kubelet-test.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubelet-test
spec:
  containers:
  - name: alpine
    image: alpine
    command: ["/bin/sh", "-c"]
    args: ["while true; do echo HanWha; sleep 15; done"]
EOF

docker ps
docker logs <Docker ID>

## etcd 설치 및 실행
```bash
wget https://github.com/etcd-io/etcd/releases/download/v3.2.26/etcd-v3.2.26-linux-amd64.tar.gz
tar -xzf etcd-v3.2.26-linux-amd64.tar.gz
pwd
ls
mv etcd-v3.2.26-linux-amd64/etcd /usr/bin/etcd
mv etcd-v3.2.26-linux-amd64/etcdctl /usr/bin/etcdctl
```
### etcd 실행
```bash
etcd --listen-client-urls http://0.0.0.0:2379 --advertise-client-urls http://localhost:2379 &> /etc/kubernetes/etcd.log &
etcdctl cluster-health
```

## kube-apiserver 실행
```bash
kubectl get all --all-namespaces
```
### kube-apiserver 동작 확인
```bash
kube-apiserver --etcd-servers=http://localhost:2379 --service-cluster-ip-range=10.0.0.0/16 --bind-address=0.0.0.0 --insecure-bind-address=0.0.0.0 &> /etc/kubernetes/apiserver.log &
ps -au | grep apiserver
head /etc/kubernetes/apiserver.log
```
### cluster context 설정
```bash
curl http://localhost:8080/api/v1
kubectl cluster-info
kubectl config set-cluster hanwha --server=http://localhost:8080
kubectl config view
kubectl config set-context hanwha --cluster=hanwha
kubectl config view
kubectl config use-context hanwha
kubectl config view
kubectl get all --all-namespaces
kubectl get nodes
```
## kubeconfig를 참조하여 kubelet 동작 확인
### kubelet 재실행
```bash
ps -ef|grep kubelet
kill -9 5066
ps -ef|grep kubelet
kubelet --register-node --kubeconfig=".kube/config" &> /etc/kubernetes/kubelet.log &
ps -au | grep kubelet
head /etc/kubernetes/kubelet.log
kubectl get nodes
```
### manifests 파일 확인
```bash
docker ps
ls /etc/kubernetes/manifests

cat <<EOF > ./kube-test.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-test
  labels:
    app: kube-test
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - name:  http
      containerPort: 80
      protocol: TCP
EOF
```
### kubelet 동작 확인
```bash
kubectl create -f kube-test.yaml
kubectl get pod
kubectl get nodes

ps -au|grep kubelet
ls -al ../.kube/config
cat ../.kube/config
kubectl describe po kube-test
```

## kube-scheduler 동작 확인
### kube-scheduler 실행
```bash
kube-scheduler --master=http://localhost:8080/ &> /etc/kubernetes/scheduler.log &
ps -au | grep scheduler
head /etc/kubernetes/scheduler.log
kubectl get pod
kubectl delete pod --all
cat <<EOF > ./replica-test.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: replica-test
spec:
  replicas: 3
  selector:
    matchLabels:
      app: replica-test
  template:
    metadata:
      name: replica-test
      labels:
        app: replica-test
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - name:  http
          containerPort: 80
          protocol: TCP
EOF
```
### deployment 실행
```bash
kubectl create -f replica-test.yaml
kubectl get po
kubectl get all
```
## kube-controller-manager 
```bash
kube-controller-manager --master=http://localhost:8080 &> /etc/kubernetes/controller-manager.log &
kubectl get deploy
kubectl get all
kubectl rollout resume deploy/replica-test
kubectl get all
kubectl rollout status deploy/replica-test
```

## pod network 확인
```bash
cat <<EOF > ./service-test.yaml
apiVersion: v1
kind: Service
metadata:
  name: replica-test
spec:
  type: ClusterIP
  ports:
  - name: http
    port: 80
  selector:
    app: replica-test
EOF

kubectl create -f service-test.yaml
kubectl get svc
curl 10.0.15.189:80
kube-proxy --master=http://localhost:8080/ &> /etc/kubernetes/proxy.log &
curl 10.0.15.189:80
```
