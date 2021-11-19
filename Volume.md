# State Persistence

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a Volume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/)

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

## Define volumes 

### 1. 아래의 내용으로 2개의 container가 있는 pod를 생성하고 2번째 container의 /etc/foo directory에 /etc/passwd file을 복사합니다. 그리고 1번째 container에서 /etc/foo directory에서 file 목록을 확인합니다.

```text
pod name: busybox
pod image: busybox
container name: box1
  run command: sleep 3600
  mount emptyDir: /etc/foo
container name: box2
  run command: sleep 3600
  mount emptyDir: /etc/foo
```

<details><summary>show</summary>
<p>
Easiest way to do this is to create a template pod with:

```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'sleep 3600' > pod.yaml
vi pod.yaml
```
Copy paste the container definition and type the lines that have a comment in the end:

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: box1
    resources: {}
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: box2 # don't forget to change the name during copy paste, must be different from the first container's name!
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  volumes: #
  - name: myvolume #
    emptyDir: {} #
```

Connect to the second container:

```bash
kubectl exec -it busybox -c box2 -- /bin/sh
cat /etc/passwd | cut -f 1 -d ':' > /etc/foo/passwd 
cat /etc/foo/passwd # confirm that stuff has been written successfully
exit
```

Connect to the first container:

```bash
kubectl exec -it busybox -c box1 -- /bin/sh
mount | grep foo # confirm the mounting
cat /etc/foo/passwd
exit
kubectl delete po busybox
```

</p>
</details>

### 2. 아래와 같이 Persistent Volume object를 생성하고 확인합니다.

```
PersistentVolume name : myvolume
PersistentVolume size : 1Gi
PersistentVolume mode : ReadWriteOnce, ReadWriteMany
StorageClassName: normal
```

<details><summary>show</summary>
<p>

```bash
vi pv.yaml
```

```YAML
kind: PersistentVolume
apiVersion: v1
metadata:
  name: myvolume
spec:
  storageClassName: normal
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  hostPath:
    path: /etc/foo
```

Show the PersistentVolumes:

```bash
kubectl create -f pv.yaml
# will have status 'Available'
kubectl get pv
```

</p>
</details>

### 3. 아래와 같이 storageClassName을 사용하여 mypvc 이름으로 persistent volume claim object를 생성하고 확인합니다.

```
PersistentVolumeClaim name : mypvc
PersistentVolumeClaim size : 1Gi
PersistentVolumeClaim mode : ReadWriteOnce, ReadWriteMany
StorageClassName: normal
```



<details><summary>show</summary>
<p>

```bash
vi pvc.yaml
```

```YAML
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mypvc
spec:
  storageClassName: normal
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Create it on the cluster:

```bash
kubectl create -f pvc.yaml
```

Show the PersistentVolumeClaims and PersistentVolumes:

```bash
kubectl get pvc # will show as 'Bound'
kubectl get pv # will show as 'Bound' as well
```

</p>
</details>

### 4. 아래와 같이 pod를 생성하고 pvc를 이용하여 /etc/foo directory에 mount하고 /etc/passwd file을 복사합니다. 

```
pod name: busybox
pod image: busybox
  run command: sleep 3600
  mount emptyDir: /etc/foo
```



<details><summary>show</summary>
<p>

Create a skeleton pod:

```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'sleep 3600' > pod.yaml
vi pod.yaml
```

Add the lines that finish with a comment:

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes: #
  - name: myvolume #
    persistentVolumeClaim: #
      claimName: mypvc #
status: {}
```

Create the pod:

```bash
kubectl create -f pod.yaml
```

Connect to the pod and copy '/etc/passwd' to '/etc/foo/passwd':

```bash
kubectl exec busybox -it -- cp /etc/passwd /etc/foo/passwd
```

</p>
</details>

### 5. 아래와 같이 2번째 pod를 생성하고(위에서 생성한 YAML file의 name을 busybox2로 변경) /etc/foo directory에 passwd file을 확인합니다. 만약 passwd file이 없다면 그 이유와 문제를 제시합니다.
```
pod name: busybox2
pod image: busybox
  run command: sleep 3600
  mount emptyDir: /etc/foo
```


<details><summary>show</summary>
<p>

Create the second pod, called busybox2:

```bash
vim pod.yaml
# change 'metadata.name: busybox' to 'metadata.name: busybox2'
kubectl create -f pod.yaml
kubectl exec busybox2 -- ls /etc/foo # will show 'passwd'
# cleanup
kubectl delete po busybox busybox2
```

보통 처음 pod를 생성하고 다음 pod를 생성하면 다른 node에 스케쥴링 됩니다.

```bash
# check which nodes the pods are on
kubectl get po busybox -o wide
kubectl get po busybox2 -o wide
```

두개의 pod가 서로 다른 node에 스케쥴링 되면 여기서는  `hostPath` volume type을 사용하므로 node 간 서로 공유 될수 없습니다.
다른 node에서도 pod가 data를 공유 하려면 공유 스토리지(NFS, Glusterfs 등)를 사용하여 volume을 구성하여야 합니다.

</p>
</details>
