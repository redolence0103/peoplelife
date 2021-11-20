# docker to containerd
![docker-process-docker-shim drawio](https://user-images.githubusercontent.com/90162116/142732876-af290a64-056d-48e1-a70a-27aa879cb4d1.png)
### node 확인
```
kubectl get nodes -o wide
NAME                 STATUS   ROLES                  AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION      CONTAINER-RUNTIME
kind-control-plane   Ready    control-plane,master   2d11h   v1.21.1   172.18.0.2    <none>        Ubuntu 21.04   5.11.0-40-generic   docker://19.3.10
kind-worker          Ready    <none>                 2d11h   v1.21.1   172.18.0.4    <none>        Ubuntu 21.04   5.11.0-40-generic   docker://19.3.10
kind-worker2         Ready    <none>                 2d11h   v1.21.1   172.18.0.3    <none>        Ubuntu 21.04   5.11.0-40-generic   docker://19.3.10
kind-worker3         Ready    <none>                 2d11h   v1.21.1   172.18.0.5    <none>        Ubuntu 21.04   5.11.0-40-generic   docker://19.3.10

```
### node container runtime 변경
```
kubectl cordon kind-worker2
kubectl drain kind-worker2 --ignore-daemonsets
ssh root@172.18.0.3
  which docker
  docker ps
  which ctr
  ctr namespace list
  NAME LABELS
  moby
  ctr --namespace moby container list
  systemctl stop kubelet
  systemctl stop docker
  apt remove --purge docker.io
  ctr namespace list
  ctr --namespace moby container list
  dpkg -L containerd.io | grep toml
  /etc/containerd/config.toml
  /user/share/man5/containerd-config.toml.5.gz
  vi /etc/containerd/config.toml
  ....
  # disabled_plugins = ["cri"]
  ....
  systemctl restart containerd
  # 2가지 항목 추가
  vi /var/lib/kubelet/kubeadm-flags.env
  KUBELET_KUBEADM_ARGS="... --container-runtime=remote --container-runtime-endpoint=unix:///run/containerd/containerd.sock ..."
  :wq!
  systemctl restart kubelet

```
### container runtime이 변경 된것을 확인합니다.
```
kubectl get nodes -o wide
NAME                 STATUS   ROLES                  AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION      CONTAINER-RUNTIME
kind-control-plane   Ready    control-plane,master   2d11h   v1.21.1   172.18.0.2    <none>        Ubuntu 21.04   5.11.0-40-generic   docker://19.3.10
kind-worker          Ready    <none>                 2d11h   v1.21.1   172.18.0.4    <none>        Ubuntu 21.04   5.11.0-40-generic   docker://19.3.10
kind-worker3         Ready    <none>                 2d11h   v1.21.1   172.18.0.3    <none>        Ubuntu 21.04   5.11.0-40-generic   docker://19.3.10
kind-worker2         Ready,SchedulingDisabled        2d11h   v1.21.1   172.18.0.5    <none>        Ubuntu 21.04   5.11.0-40-generic   containerd://1.4.3
```
###  node를 활성화하고 동일하게 다른 node에 진행하며 wokernode 완료후 controlnode를 동일하게 진행합니다.
```
kubectl uncordon kind-woker2
```
