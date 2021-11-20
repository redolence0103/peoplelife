# Upgrade K8S

### kubeadm으로 K8S version 1.19.0 에서 1.20.0으로 upgrade 합니다.

1. control plane을 upgrade 합니다.

```bash
# 현재 version을 확인합니다.
kubectl version --short
Client Version: v1.19.0
Server Version: v1.19.0
kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.0", GitCommit:"e19964183377d0ec2052d1f1fa930c4d7575bd50", GitTreeState:"clean", BuildDate:"2020-08-26T14:28:32Z", GoVersion:"go1.15", Compiler:"gc", Platform:"linux/amd64"}

# package를 확인하고 설정합니다.
apt update
apt-cache madison kubeadm |grep -i 1.20.0
....
kubeadm |  1.20.0-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
....
apt-mark showhold
....
kubeadm
kubectl
kubelet
....
apt-mark unhold kubeadm kubectl kubelet
# kubeadm 설치
apt install -y kubeadm=1.20.0-00
# kubeadm upgrade
kubeadm upgrade plan
....
COMPONENT                 CURRENT   AVAILABLE
kube-apiserver            v1.19.0   v1.20.12
kube-controller-manager   v1.19.0   v1.20.12
....
kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.0", GitCommit:"af46c47ce925f4c4ad5cc8d1fca46c7b77d13b38", GitTreeState:"clean", BuildDate:"2020-12-08T17:57:36Z", GoVersion:"go1.15.5", Compiler:"gc", Platform:"linux/amd64"}

kubeadm upgrade apply v1.20.0
....
[upgrade/confirm] Are you sure you want to proceed with the upgrade? [y/N]: y
[upgrade/prepull] Pulling images required for setting up a Kubernetes cluster
[upgrade/prepull] This might take a minute or two, depending on the speed of your internet connection
[upgrade/prepull] You can also perform this action in beforehand using 'kubeadm config images pull'
....
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.20.0". Enjoy!
[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
apt install -y kubelet=1.20.0-00 kubectl=1.20.0-00
# 시스템에 적용
systemctl daemon-reload
systemctl restart kubelet
# worker node drain
kubectl drain node01 --ignore-daemonsets --force
# 해당 node에 running 중인 pod가 다른 node로 이동 된것을 확인합니다.
kubectl get pod -A -o wide
kubectl get nodes


```



### kubeadm으로 node를 추가합니다.

```bash
# node에 workload 스케쥴링 제외
kubectl drain kworker1 --ignore-daemonset --force
kubeadm token list
TOKEN                     TTL         EXPIRES                USAGES                   DESCRIPTION                     EXTRA GROUPS
abcdef.0123456789abcdef   15h         2021-11-18T02:07:38Z   authentication,signing   <none>                         system:bootstrappers:kubeadm:default-node-token
# 만약 없다면 token 생성
kubeadm token create
# hash 값 생성
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex
(stdin)= 913a5448ea28a47e33e8ee289d79ed9cace51a5a3c8b0191eb3463e01a268402
# login into kworker1
kubeadm join https://172.18.0.2:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:913a5448ea28a47e33e8ee289d79ed9cace51a5a3c8b0191eb3463e01a268402
# node가 성공적으로 추가되면 스케쥴링 허용합니다.
kubectl uncordon kworker1
```
