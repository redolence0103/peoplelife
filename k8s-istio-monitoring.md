# Istio and Prometheus and Grafana
https://istio.io/latest/docs/
### Install Istio

```bash
curl -L https://istio.io/downloadIstio | sh -
export PATH=$PATH:/home/atid/istio-1.11.4/bin
istioctl install
```
### Install Demo Application

```bash
kubectl label namespace demo istio-injection=enabled
namespace/demo labeled
kubectl label namespace demo istio-injection=enabled --overwrite
kubectl config set-context --current --namespace=demo
# app 배포
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
# istio-gateway 배포
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```


### Install Prometheus and Grafana

```bash
# helm version 3 이 설치 되어있어야합니다.
# prometheus and grafana helm chart를 등록합니다.
helm repo add starble https://charts.helm.sh/stable
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm search repo prometheus-community
# 설치할 namespace를 생성합니다.
kubectl create namespace prometheus
kubectl create namespace grafana
# persistentVolume 정보를 변경하고 설치합니다.
helm inspect values life prometheus-community/prometheus > /tmp/prometheus.yaml
# 2개 부분의 persistentVolume을 활성화하고 pvc를 확인합니다.
vi /tmp/prometheus.yaml
.....
  persistentVolume:
    ## If true, alertmanager will create/use a Persistent Volume Claim
    ## If false, use emptyDir
    ##
    enabled: true
.....
:wq!
helm install life prometheus-community/prometheus --values /tmp/prometheus.yaml --namespace=prometheus
kubectl get pvc -n prometheus

helm inspect values life bitnami/grafana > /tmp/grafana.yaml
vi /tmp/grafana.yaml
.....
persistence:
  enabled: true
  ## If defined, storageClassName: <storageClass>
.....
:wq!
helm install life bitnami/grafana --values /tmp/grafana.yaml --namespace=grafana
kubectl get pvc -n grafana
```



### WEB Service 접속

```bash
# External or NodePort를 사용하여 web으로 접속합니다.
kubectl get svc -n grafana
NAME           TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
atid-grafana   LoadBalancer   10.96.16.230   172.18.70.2   3000:30197/TCP   3h32m

kubectl get svc -n prometheus
NAME                            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
atid-kube-state-metrics         ClusterIP      10.96.201.194   <none>        8080/TCP       3h2m
atid-prometheus-alertmanager    ClusterIP      10.96.47.221    <none>        80/TCP         3h2m
atid-prometheus-node-exporter   ClusterIP      None            <none>        9100/TCP       3h2m
atid-prometheus-pushgateway     ClusterIP      10.96.37.41     <none>        9091/TCP       3h2m
atid-prometheus-server          LoadBalancer   10.96.83.192    172.18.70.1   80:31724/TCP   3h2m
```
