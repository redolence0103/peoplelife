# NFS for K8S

### nfs server, client 설치
```bash
sudo apt update
# nfs client만 설치시 nfs-common만 설치합니다.
sudo install -y nfs-kernel-server nfs-common
```
### nfs server 설정
``` bash
# service 대상 directory를 확인하고 /etc/exports 파일의 맨 마지막 라인을 적절한 옵션으로 추가합니다.
vi /etc/exports
# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/data/nginx       *(rw,sync,no_subtree_check)
wq!

sudo systemctl restart nfs-kernel-server
# 서버 내부에서 nfs service를 확인합니다.
exportfs 
/data/nginx     <world>
# 외부에서 nfs service를 확인합니다.
showmount -e kibana
Export list for kibana:
/data/nginx *
```
### nfs-subdir-external-provisioner 설치
```bash
# 먼저 각 node에 nfs client가 설치되어야합니다.
# x.x.x.x 는 nfs server ip, /exported/path 는 nfs export list에 있는 path로 변경
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=x.x.x.x \
    --set nfs.path=/exported/path
```
