# kubectl port-forward 사용법

## ingress 외부 Test 방법
- client에서 hosts file에 '218.236.22.95 life.com' 등록 (windows: C:\Windows\System32\drivers\etc\hosts)
```
atid@kibana:~$ kubectl -n ingress-nginx port-forward --address 218.236.22.95 service/nginx-ingress-ingress-nginx-controller 5762:80
Forwarding from 218.236.22.95:5762 -> 80
```
## Pod 외부 Test 방법
- client에서 http://218.236.22.95:5762
```
atid@kibana:~$ k port-forward --address 218.236.22.95 pod/jenkins-7b57496c88-qbbxc 5762:80
Forwarding from 218.236.22.95:5762 -> 80
```
## Service 외부 Test 방법
- client에서 http://218.236.22.95:5762
```
atid@kibana:~$ k port-forward --address 218.236.22.95 service/jenkins 5762:80
Forwarding from 218.236.22.95:5762 -> 80
```
## iptables를 사용한 Test 방법
- cluster에 접속된 kubectl 환경에서
```
iptables -t nat -A PREROUTING -p TCP --dport 5762 -j DNAT --to-destination 172.18.70.4:80
iptables -A FORWARD -p tcp --dport 80 -d 172.18.70.4 -j ACCEPT

## kind apiserver.crt에 Alternative Name 추가 방법
외부의 kubectl이 설치된 환경에서 접속가능한 IP 대역(공인 또는 Local)에서 실행 중인 KIND K8S 클러스터에 접근 방법
- KIND cluster에 접속된 kubectl 환경에서 실행
```bash
# KIND Cluster 실행 서버 IP POrtForwarding (iptables 사용)
# apiserver.crt에 등록된 alternative name 확인
# 실행 서버 IP와 PortForwarding 포트 등록 (접근할 kubectl이 설치된 terminal)
openssl x509 -in apiserver.crt -text
kubeadm init phase certs apiserver --apiserver-cert-extra-sans=218.236.22.90
kubectl -n kube-system get configmap kubeadm-config -o yaml
kubectl -n kube-system edit configmap kubeadm-config -o yaml # IP list 추가(218.236.22.90)

```
