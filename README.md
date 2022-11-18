# peoplelife K8S 학습 환경 설치

- MSA 실습 web: https://labs.msaez.io/

peoplelife  교육자료
Linux vm(ubuntu 20.04 lts)에 KIND 설치 내역
- https://kind.sigs.k8s.io/
- https://www.youtube.com/watch?v=G0EW8QLwjwc&list=PLEr96Fo5umW9Gf-Ej6FF5G-QbBBvyrCoQ&index=6
- https://www.youtube.com/watch?v=PqWJcpIdyV4&list=PLEr96Fo5umW9Gf-Ej6FF5G-QbBBvyrCoQ&index=7

![kind-k8s-env](https://user-images.githubusercontent.com/90162116/142726818-9f0747b5-04e5-42d4-881e-10a99c52d13c.png)

## vm 서버에 docker 설치 

```text
sudo sysctl -w net.ipv4.ip_forward=1
sudo swapoff -a
sudo apt update -y
sudo apt install -y docker.io
sudo usermod -aG docker $USER
```

## kubectl 설치

```bash
curl -LO https://dl.k8s.io/release/v1.21.0/bin/linux/amd64/kubectl
```



## kind 설치 및 docker 기반 k8s cluster 생성

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

cat << EOF > ./kind.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
EOF

kind create cluster --config kind.yaml
```



## cluster에 matrix server 설치. 

matrix server download

```bash
wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.5.0/components.yaml
```

matrix server 와 kubelet과의 비보안 통신 설정

```bash
vi components.yaml
  .....
        args:
          - --cert-dir=/tmp
          - --secure-port=4443
          - --kubelet-insecure-tls
  .....
:wq!

kubectl create -f components.yaml
```

## metallb 설치 및 설정. 

kube-proxy를 ipvs mode로 설정

```
kubectl -n kube-system edit configmap kube-proxy
  .....
mode: "ipvs"
ipvs:
  strictARP: true
  .....
```


metallb 설치

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/metallb.yaml
```

metallb Layer2 설치

```YAML
cat << EOF > ./config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 172.18.70.1-172.18.70.254
EOF
```

```bash
kubectl apply -f config.yaml
```



## helm 설치 (version 3)

```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```



## ingress-nginx 설치 (heml)

```
helm repo add stable https://charts.helm.sh/stable
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm repo list
helm show values ingress-nginx/ingress-nginx > /tmp/ingress-nginx.yaml
```

hostNetwork 사용을 위한 config 설정

```bash
vi /tmp/ingress-nginx.yaml
  .....
hostNetwork: True
  .....
  hostPort:
    enabled: True
  .....
kind: DaemonSet
  .....
:wq!
```

ingress-nginx 설치

```
kubectl create ns ingress-nginx
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace=ingress-nginx --values /tmp/ingress-nginx.yaml
```
