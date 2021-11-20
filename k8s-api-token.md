# Token을 이용한 Service 통신

### 먼저 token 확인을 위한 jwt-cli 설치합니다.

```bash
apt install -y npm
npm install -g jwt-cli
```



### image=nginx를 사용하여 pod를 생성하고 pod의 serviceAccountName을 확인합니다.

<details><summary>show</summary>
<p>

```
kubectl run nginx --image=nginx
kubectl get pod nginx -o yaml
....
  serviceAccount: default
  serviceAccountName: default
....
```

</p>
</details>

### 위에서 확인된 default 로 명명된 serviceAccount와 함께 생성된 secret을 확인합니다.

<details><summary>show</summary>
<p>

```html
kubectl get sa default -o yaml
kubectl get secret default-token-t7m7z  -o yaml
```

</p>
</details>

###  위에서 확인된 token으로 nginx에서 kube-apiserver로 통신을 요청합니다.

<details><summary>show</summary>
<p>

```bash
kubectl exec -it nginx -- /bin/bash
/# ls /var/run/secrets/kubernetes.io/serviceaccount/
ca.crt namespace token
/# cat ca.crt
/# cat token
/# cd /var/run/secrets/kubernetes.io/serviceaccount/
/# CA=$PWD/ca.crt
/# TOKEN=$(cat $PWD/token)
/# curl --cacert $CA -X GET https://kubernetes/api --header "Authorization: Bearer $TOKEN"
/# curl -X GET https://kubernetes/api --header "Authorization: Bearer $TOKEN" --insecure
```

</p>
</details>

### 설치된 jwt로 token을 확인합니다.

<details><summary>show</summary>
<p>

```bash
jwt eyJhbGci..........f6f2g
......
✻ Payload
{
  "iss": "kubernetes/serviceaccount",
  "kubernetes.io/serviceaccount/namespace": "istio-system",
  "kubernetes.io/serviceaccount/secret.name": "default-token-kb2zq",
  "kubernetes.io/serviceaccount/service-account.name": "default",
  "kubernetes.io/serviceaccount/service-account.uid": "74cc6297-7fb0-4452-82b3-4adf37e7b87d",
  "sub": "system:serviceaccount:istio-system:default"
}
......
```

</p>
</details>
