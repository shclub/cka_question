# CKA 문제


<br/>

## 1. ETCD Backup & Restore  

<br/>


> Question : ETCD Backup & Restore  

> TASK :  
>  - First, create a snapshop of the existing etcd instance  running at `https://127.0.0.1:2379`, saving the snapshot to `/data/etcd-snapshot.db`.
>  - Next, restore and existing, previous snapshot  located at `/data/etcd-snapshot-previous.db`.  

>  The following TLS certificates/key are supplied for connecting to the server with etcdctl:  
>  - CA certificate : /etc/kubernetes/pki/etcd/ca.crt     
>  - Client certificate : /etc/kubernetes/pki/etcd/server.crt     
>  - Client key : /etc/kubernetes/pki/etcd/server.key       


<br/>

## 2. pod 생성하기  


<br/>

> Question : Create a new namespace and create a pod in the namespace 

> TASK :  
>  - namespace name : edu30  
>  - pod name : eshop-main 
>  - image : nginx:1.17 
>  - env : DB=mysql 


<br/><br/>


> 결과값  

```bash
root@newedu:~# kubectl exec -it eshop-main   -- printenv
DB=mysql
```  

<br/>

## 3. Static POD 생성하기

<br/>

Master Node의 API를 호출 하지 않아 Scheduler 가 생성하지 않는 POD 이며 각 Node의 kubelet ( Daemon ) 이 생성한다.  

<br/>

k8s에는 /var/lib/kubelet/config.yaml 에서 static POD yaml 화일을 위치를 볼수 있고 일반적으로 /etc/kubernetes/manifests 폴더에 있다.  

<br/>

Master Node 의 POD 들도 Static POD 이고 kubelet 에 의해서 기동된다.  

<br/>

> Question : Configure kubelet hosting to `start a pod on > the node`  

> TASK :  
>  - Node : edu.worker05
>  - Pod Name : web
>  - image : nginx

<br/><br/>


> 결과값  

 ```bash
 root@newedu:~# kubectl get po -n default
 default                                            web-edu.worker05                                                  1/1     Running                0          2m54s
```  


<br/>

## 4. Multi Container POD 생성하기

<br/>

하나의 POD에 Container를 여러개 생성 할 수 있다.  

<br/>

> Question : Create Pod 

> TASK :  
>  - Create a pod name multi-con with 3 containers running : nginx, redis.memcached  

<br/><br/>


> 결과값  

```bash
root@newedu:~# kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
multi-con                           3/3     Running   0          31s
root@newedu:~# kubectl describe po multi-con
Name:         multi-con
Namespace:    edu30
Priority:     0
Node:         edu.worker02/172.25.1.50
Start Time:   Fri, 23 Sep 2022 20:47:25 +0900
Labels:       run=multi-con
...
Containers:
  nginx:
    Container ID:   cri-o://8594ed5595f6d8049fd8368473706d147fc1d48ebd2fc53f338393d97de35cdd
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:79c77eb7ca32f9a117ef91bc6ac486014e0d0e75f2f06683ba24dc298f9f4dd4
    Port:           <none>
...
  redis:
    Container ID:   cri-o://56b3d75c4491df928ef66d1077077c2e3c0bcc12559e62a8b220b1f4bcc46c01
    Image:          redis
    Image ID:       docker.io/library/redis@sha256:b4e56cd71c74e379198b66b3db4dc415f42e8151a18da68d1b61f55fcc7af3e0
...
  memcached:
    Container ID:   cri-o://0e11b71e73894b5b69bba208eb7dc9c669dc635af982fe8121b02806f50ab5e3
    Image:          memcached
    Image ID:       docker.io/library/memcached@sha256:324479339ef09ec98467d7c0a8083909503e7af3095390614990af0fee781c7b
...
  Normal  Pulling         2m26s  kubelet            Pulling image "nginx"
  Normal  Pulled          2m22s  kubelet            Successfully pulled image "nginx" in 4.018533684s
  Normal  Created         2m22s  kubelet            Created container nginx
  Normal  Started         2m22s  kubelet            Started container nginx
  Normal  Pulling         2m22s  kubelet            Pulling image "redis"
  Normal  Pulled          2m12s  kubelet            Successfully pulled image "redis" in 10.541276692s
  Normal  Started         2m11s  kubelet            Started container redis
  Normal  Pulling         2m11s  kubelet            Pulling image "memcached"
  Normal  Created         2m11s  kubelet            Created container redis
  Normal  Pulled          2m2s   kubelet            Successfully pulled image "memcached" in 9.577820141s
  Normal  Created         2m2s   kubelet            Created container memcached
  Normal  Started         2m2s   kubelet            Started container memcached
```  


<br/>


## 5. Side-car container POD 실행

<br/>

기존 POD에 로그 분석을 위한 별도 Container 를 Side Car 로 추가할 수 있다.

<br/>

> Question : An existing Pod needs to be integrated into the K8s built-in logging architecture (e.g. `kubectl logs` )  
> Adding a streaming `sidecar` container is good and common way tyo accomplish this requirement.  

> TASK :  
>  - Add a sidecard container named `sidecar`, using `busybox` image, to the existing Pod `eshop-cart-app`.  
>  - The new sidecar container has to run the following command: /bin/sh -c 'tail -n +1 -F /var/log/eshop-app.log'  
>  - Use a Volume mounted at `/var/log`, to make the log file `eshop-cart-app.log` avaiable to the sidecar container.  
>  - `Don't modify the eshop-cart-app`

<br/><br/>

일단 기동 중인  eshop-cart-app 라는 pod에서 yaml 화일을 받아 옵니다.  
( 우리는 현재 기동한 POD가 없기 떼문에 아래 yaml를 사용한다. )  

- kubectl get po eshop-cart-app -o yaml > eshop.yaml  

<br/>

eshop.yaml
```bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: eshop-cart-app
  name: eshop-cart-app
spec:
  containers:
  - command:
    - /bin/sh
    - -c
    - 'i=1;while :;do echo -e "$i: price: $((RANDOM % 10000 +1))" >> /var/log/cart-app.log; i=$((i+1)); sleep 2;done'
    image: busybox
    name: eshop-cart-app
    volumeMounts:
    - mountPath: /var/log
      name: varlog
volumes:
  - emptyDir: {}
    name: varlog
```  

> 결과값  

```bash
root@newedu:~# kubectl get po 
NAME                                READY   STATUS    RESTARTS   AGE
eshop-cart-app                      2/2     Running   0          11s
```  

<br/>

sidecar 컨테이너에서  로그 정보가 나와야 한다.  

```bash
root@newedu:~# kubectl logs eshop-cart-app -c sidecar
1: price: 9448
2: price: 5839
3: price: 7135
4: price: 9062
5: price: 2709
6: price: 348
7: price: 9538
8: price: 356
9: price: 5786
...
```  

<br/>



## 6. Deployment & Pod Scale

<br/>

### 1. Pod scale out

<br/>

> Question : Expand the number of running Pods in `eshop-order` to `5`.

> TASK :  
>  - namespace : 본인 namespace  
>  - deployment : eshop-order

<br/><br/>

아래 yaml 화일을 실행하여 deployment 를 생성합니다.  
- root@newedu:~# kubectl create deployment eshop-order --image=nginx --replicas=1 

<br/>

```bash
root@newedu:~# kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
eshop-order        1/1     1            1           7s
```  

> 결과값  : pod가 5개가 생성되었는지 확인한다.

```bash
root@newedu:~# kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
eshop-order        5/5     5            5           46s
root@newedu:~# kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
eshop-order-ff866c56b-8k8cq         1/1     Running   0          14s
eshop-order-ff866c56b-dcfr9         1/1     Running   0          14s
eshop-order-ff866c56b-fh644         1/1     Running   0          14s
eshop-order-ff866c56b-ltkg6         1/1     Running   0          58s
eshop-order-ff866c56b-q9gd2         1/1     Running   0          14s
```  


<br/>


### 2. Deployment 생성하고 Scaling 하기

<br/>

> Question : Create a deployment as follows. 

> TASK :  
>  - name : `webserver` 
>  - `2` replicas
>  - label : `app_env_stage=dev`
>  - container name : `webserver`
>  - container image : `nginx:1.14`  


>  Scale Out Deployment.  
>  - Scale the deployment webserver to `3` pods

<br/><br/>

아래 yaml 화일을 실행하여 deployment 를 생성합니다.  
- root@newedu:~# kubectl create deployment webserver --image=nginx:1.14 --replicas=2 -l app_env_stage=dev

<br/>

```bash
root@newedu:~# kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
eshop-order        1/1     1            1           7s
```  

> 결과값  : webserver 이름으로 deployment 생성 ( Replica : 2 )

```bash
root@newedu:~# kubectl get deploy
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
webserver          2/2     2            2           5m29s
```  

<br/>

replicas 3 으로 증가.  

```bash
root@newedu:~# kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
webserver-5586594bbf-kqkfr          1/1     Running   0          3m47s
webserver-5586594bbf-td2z5          1/1     Running   0          3m47s
webserver-5586594bbf-vpf5j          1/1     Running   0          7s
```  

<br/>


## 7. Rolling Update

<br/>

> Question : Create a deployment as follows. 

> TASK :  
>  - name : `nginx-app`  
>  - Using container nginx with version : `nginx:1.14-alpine`
>  - The deployment should contain `3` replicas
>  Next, deploy the application with new version `nginx:1.14.2-alpine`, by performing a rolling update   
>  Finally, rollback that update to the previous version `nginx:1.14-alpine`

<br/><br/>

> 결과값  : deployment 와 rollout history를 확인 합니다.  


```bash
root@newedu:~# kubectl rollout  history deployment nginx-app
deployment.apps/nginx-app
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
```  


<br/>



## 8. Node Selector

<br/>

> Question : Schedule a pod as follows. 

> TASK :  
>  - name : `eshop-store`  
>  - image : `nginx`
>  - Node Selector :  `disktype=ssd`  

<br/><br/>

> 결과값  : node의 label를 확인 하고 pod 생성.  


node별로 disktype label 확인.  


```bash
root@newedu:~# kubectl get nodes -L disktype 
NAME               STATUS   ROLES    AGE   VERSION                DISKTYPE
edu.dmz-infra01    Ready    worker   97d   v1.20.0+bafe72f-1054
edu.dmz-infra02    Ready    worker   97d   v1.20.0+bafe72f-1054
edu.master01       Ready    master   98d   v1.20.0+bafe72f-1054
edu.master02       Ready    master   98d   v1.20.0+bafe72f-1054
edu.master03       Ready    master   98d   v1.20.0+bafe72f-1054
edu.monitoring01   Ready    worker   97d   v1.20.0+bafe72f-1054
edu.monitoring02   Ready    worker   97d   v1.20.0+bafe72f-1054
edu.worker01       Ready    worker   97d   v1.20.0+bafe72f-1054
edu.worker01       Ready    worker   97d   v1.20.0+bafe72f-1054
...
edu.worker07       Ready    worker   97d   v1.20.0+bafe72f-1054   ssd
...
```  

<br/>

pod가 특정 node에서 실행이 됨.  

```bash
root@newedu:~# kubectl get po eshop-store -o wide
NAME          READY   STATUS    RESTARTS   AGE   IP             NODE           NOMINATED NODE   READINESS GATES
eshop-store   1/1     Running   0          16s   10.131.6.100   edu.worker07   <none>           <none>
```  


<br/>



## 9. Node 관리

<br/>

cordon : 해당 노드에 pod scheduling 불가.

<br/>

```bash
root@newedu:~# kubectl get  nodes
NAME               STATUS   ROLES    AGE   VERSION
...
edu.worker01       Ready    worker   97d   v1.20.0+bafe72f-1054
edu.worker02       Ready    worker   97d   v1.20.0+bafe72f-1054
edu.worker03       Ready    worker   97d   v1.20.0+bafe72f-1054
...
root@newedu:~# kubectl cordon edu.worker01
node/edu.worker01 cordoned
root@newedu:~# kubectl get  nodes
NAME               STATUS                     ROLES    AGE   VERSION
edu.worker01       Ready,SchedulingDisabled   worker   97d   v1.20.0+bafe72f-1054
edu.worker02       Ready                      worker   97d   v1.20.0+bafe72f-1054
edu.worker03       Ready                      worker   97d   v1.20.0+bafe72f-1054
```  

<br/>

uncordon : 해당 노드에 pod scheduling 가능.

<br/>

drain : 해당 노드의 모든 pod를 다른 node로 보내고 현재 노드에 스케쥴링 하지 말아라

```bash
root@newedu:~# kubectl drain edu.worker07 --ignore-daemonsets
node/edu.worker07 cordoned
error: unable to drain node "edu.worker07", aborting command...

There are pending nodes to be drained:
 edu.worker07
error: cannot delete Pods not managed by ReplicationController, ReplicaSet, Job, DaemonSet or StatefulSet (use --force to override): edu15/hostpath-storage-pod, edu30/eshop-store, edu9/pod-test-app
```  

<br/>


위에서 보면 정상적으로 되어 있지 않을것을 볼 수 있다.
daemonset 은 죽이지 못해서 나는 에러이고 이것을 무시하는 명령어를  포함하여 다시 수행한다.  

cannot delete DaemonSet-managed Pods (use --ignore-daemonsets to ignore)

<br/>

```bash
root@newedu:~# kubectl drain edu.worker07 --ignore-daemonsets
node/edu.worker07 already cordoned
error: unable to drain node "edu.worker07", aborting command...

There are pending nodes to be drained:
 edu.worker07
error: cannot delete Pods not managed by ReplicationController, ReplicaSet, Job, DaemonSet or StatefulSet (use --force to override): edu15/hostpath-storage-pod, edu30/eshop-store, edu9/pod-test-app
root@newedu:~# kubectl drain edu.worker07 --ignore-daemonsets --force to override
```  

<br/>

정상적으로 수행이 된다.  

```bash
victing pod edu9/pod-test-app
evicting pod edu2/flask-edu4-app-757bcc87db-lzd8l
evicting pod edu30/canary-rollout-7f896b6cd8-tq8zn
...
evicting pod edu2/nginx-deployment-78558c798-xvxwl
pod/canary-rollout-7f896b6cd8-tq8zn evicted
pod/eshop-store evicted
pod/mynginx-69d586ff67-hd76b evicted
pod/dev-edu6-889589895-rw8jw evicted
pod/flask-edu4-app-757bcc87db-lzd8l evicted
pod/backend-f866cb67d-nwdth evicted
pod/flask-edu4-app-74788b6479-wmlvj evicted
node/edu.worker07 evicted
```  

<br/>

node를 조회해 본다.  


```bash
root@newedu:~# kubectl get nodes
NAME               STATUS                     ROLES    AGE   VERSION
edu.worker01       Ready,SchedulingDisabled   worker   98d   v1.20.0+bafe72f-1054
```  

<br/>

pod를 조회해 보면 1개의 pod ( canary-rollout-7f896b6cd8-kzdbs ) 가 다른 node에서 뜬 걸 볼수 있다.  

```bash
root@newedu:~# kubectl get po -n edu30
NAME                                READY   STATUS    RESTARTS   AGE
busybox-nfs-test-687b87878d-xxnbv   1/1     Running   0          4d3h
canary-rollout-7f896b6cd8-78mxr     1/1     Running   0          4d6h
canary-rollout-7f896b6cd8-8wr9k     1/1     Running   0          4d6h
canary-rollout-7f896b6cd8-blzlj     1/1     Running   0          4d6h
canary-rollout-7f896b6cd8-ddlfk     1/1     Running   0          4d6h
canary-rollout-7f896b6cd8-kzdbs     1/1     Running   0          4m11s
canary-rollout-7f896b6cd8-mlhb9     1/1     Running   0          4d6h
canary-rollout-7f896b6cd8-nmvqc     1/1     Running   0          4d6h
canary-rollout-7f896b6cd8-pkd4c     1/1     Running   0          4d6h
```  

<br/>

다시 원복 하기 위해서는 uncordon 명령어를 사용한다. 

```bash
root@newedu:~# kubectl uncordon edu.worker07
node/edu.worker07 uncordoned
root@newedu:~# kubectl get nodes
NAME               STATUS   ROLES    AGE   VERSION
...
edu.worker07       Ready    worker   18d   v1.20.0+bafe72f-1054
```  

<br/>


> Question : Set the node named  `k8s-worker1` as `unavailable` and `reschedule` all the pods running on it.  

<br/><br/>

> 결과값  : drain 명령어 사용

```bash
root@newedu:~# kubectl get nodes
NAME               STATUS                     ROLES    AGE   VERSION
k8s-worker1       Ready,SchedulingDisabled   worker   98d   v1.20.0+bafe72f-1054
```  

<br/>


## 10. Node 정보 수집

<br/>

### 1. Check Ready Nodes  

> Question : Check to see how many nodes are `ready` ( not including nodes tainted NoSchedule) and `write the number` to `./RN0001`. 


<br/><br/>

> 결과값  : /RN0001 파일을 열어 본다. 5가 나와야 함.  


```bash
root@newedu:~# cat ./RN0001
5
```  

<br/>




<br/>

### 2. Count the Number of Nodes That Are Ready to Run Normal Workloads  

> Question : Determine `how many nodes in the cluster` and `ready` to run normal workloads (i.e, workloads that do not have any special tolerations ).  

>  Output this number to the file `write the number` to `./NODE-count`. 


<br/><br/>

> 결과값  : /NODE-count 파일을 열어 본다. 14가 나와야 함.  


```bash
root@newedu:~# cat ./NODE-count
14
```  


<br/>


## 11. Deployment & Expose the Service

<br/>

> Question : Reconfigure the existing deployment `front-end` and `add a port specification named http` exposing port `80/tcp` of the  existing  container nginx.  

> Create a new service named `front-end-svc` exposing the container port `http`.  

> Configure the new service to also expose the individual Pods via a `NodePort` on the nodes on which they are scheduled.


<br/>

먼저 아래 yaml 화일을 deployment를 생성해 놓는다.    

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: front-end
  name: front-end
spec:
  replicas: 2
  selector:
    matchLabels:
      app: front-end
  template:
    metadata:
      labels:
        app: front-end
    spec:
      containers:
      - image: nginx
        name: nginx
```  

deployment 가  실행되어 있는 것을 확인한다.  

```bash
root@newedu:~# kubectl get deploy
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
front-end          2/2     2            2           8s
```
<br/><br/>

>  결과값  : svc를 조회 해 보고 서비스 이름과 NodePort가 생성되어 있는지 확인.  

<br/>


service를  조회 해 보면 NodePort가 생성된 것을 확인 할 수있다.  

```bash
root@newedu:~# kubectl get svc
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
front-end-svc               NodePort    172.30.145.50    <none>        80:32495/TCP     2m41s```  
```  

<br/>

curl 명령어로 해당 서비스를 호출해 본다.    

아래와 같이 niginx 첫 화면 내용이 나오면 성공.  

```bash
root@newedu:~# curl 211.34.231.84:32495
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```  


<br/>


## 12. Pod log 추출

<br/>

Pod Log를 stand out 으로 추출.

> Question : Monitor the logs of pod `custom-app` and Extract log lines corresponding to error `file not found`. Write them  to `./podlog`.


<br/><br/>

>  결과 및 답안  : kubectl logs로 조회한 후 저장  

<br/>


아래 명령어 사용  

```bash
root@newedu:~# kubectl logs custom-app | grep 'file not found' >  ./podlog 
```  

<br/>

## 13. CPU 사용량이 높은 Pod 검색

<br/>


> Question : From the pod label `name=overloaded-cpu`, find pods running high CPU workloads and write the name of the pod consuming most CPU the file `./cpu_load_pod.txt`.


<br/><br/>

>  결과 및 답안  : kubectl top 로 조회한 후 저장  

<br/>


아래 명령어 사용  

```bash
root@newedu:~# kubectl top po -l name=overloaded-cpu --sort-by=cpu 
root@newedu:~# echo "POD_NAME" >  ./cpu_load_pod.txt 
```  

<br/>

## 14. init 컨테이너를 포함한 Pod 운영

<br/>

init 컨테이너는 main container 가 동작 되기전에 실행되는 컨테이너
- 주로 환경 구성 . DB 인 경우는 초기 DB Script   

init 컨테이너는 항상 완료가 되어야 한다.    

참고 : https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

<br/>

> Question : Perform the following.  

> TASK :  
>  Add an init container to `web-pod`(which has been defined in spec file  `./webpod.yaml`).    
>  The init container should create an empty file named `/home/data.txt`.    
>  if `/home/data.txt` is not detected. the Pod should exit.  
>  Once the spec file has been `updated with the init container` definition. the pod should be created.  


<br/>
 
web-pod.yaml 화일  
```bash
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
  - image: busybox:1.28
    name: main
    command: ['sh','-c','if [ !-f /home/data.txt ];then exit 1;else sleep 300;fi']
    volumeMounts:
    - name: workdir
      mountPath: "/home"
  volumes:
  - name: workdir
    emptyDir: {}
```   

<br/><br/>

>  결과값  : web-pod 이름의 pod에서 /home 폴더에  data.txt 화일이 있어야 함.

<br/>


아래 명령어 사용  

```bash
root@newedu:~# kubectl exec -it web-pod sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/ # ls
bin   dev   etc   home  proc  root  run   sys   tmp   usr   var
/ # cd home
/home # ls
data.txt
```  

<br/>



## 15. NodePort 서비스 생성

<br/>


> Question : Create the service as type `NodePort` with `port 32767` for the nginx pod with pod selector `app: webui`  

<br/>
 
nginx.yaml 화일  
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: webui
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webui
  strategy: {}
  template:
    metadata:
      labels:
        app: webui
    spec:
      containers:
      - image: nginx
        name: nginx
```   

<br/><br/>

>  결과값  : 서비스를 조회 하여 해당 서비스가 NodePort 로 설정 되어 있는지 확인한다.  

<br/>

서비스를 조회한다.   

```bash
root@newedu:~# kubectl get svc  
nginx                       NodePort    172.30.228.71    <none>        80:32767/TCP     16s
```  

curl 명령어로 서비스를 호출한다.  

```bash
root@newedu:~# curl 211.34.231.84:32767
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>  
```  

<br/>


## 16. ConfigMap 운영

<br/>

configMap을 환경 변수로 전달하는 방법과 volume mount를 해서 전달하는 방법 2가지를 다 공부해야 한다.    

https://kubernetes.io/docs/concepts/configuration/configmap/

<br/>

> Question : Expose Configuration settings

> TASK :  
>  All operations in this question should be performed in the < your namespace >
>  Create `ConfigMap` called `web-config` that contains the following `two` entries
>  - `connection_string=localhost:80`
>  - `external_url=cncf.io`

>  Run a pod called `web-pod` with a single container running the `nginx:1.19.8-alpine` image, and expose these configuration settings as `environment variables` inside the container. 
  

<br/><br/>

>  결과값  : pod 안에 환경 변수 2개가 들어 있는지 확인한다.  

<br/>


해당 POD의 환경 변수를 확인한다.  

```bash
root@newedu:~# kubectl exec  web-pod -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
TERM=xterm
HOSTNAME=web-pod
connection_string=localhost:80
external_url=cncf.io
```
  

<br/>


## 17. Secret 운영

<br/>

Secret 은 configMap과 비슷하지만 base64로 인코딩 하는 부분만 다름.  

https://kubernetes.io/docs/concepts/configuration/secret/  


<br/>

Secret는 용도에 따라서 3가지가 있고 우리는 일반 적인 방식인 Generic으로 진행을 합니다.  

```bash
root@newedu:~# kubectl create secret --help
Create a secret using specified subcommand.

Available Commands:
  docker-registry Create a secret for use with a Docker registry
  generic         Create a secret from a local file, directory or literal value
  tls             Create a TLS secret

Usage:
  kubectl create secret [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
```  


<br/>

Secret을 생성 후 Pod에 전달  

> Question : Create a Kubernetes secret and expose  using a file in the pod.  

> TASK :  
>  Create a Kubernetes Secret as follows.
>  - Name : `super-secret` 
>  - DATA : `password=secretpass`  

>  Create a Pod named `pod-secrets-via-file`, using the `redis` image, which  mounts a secret named `super-secret` at `/secrets`.

>  Create a second Pod named `pod-secrets-via-env`, using the redis image, which exports `password` as `PASSWORD`.  

<br/><br/>

>  결과값  : 

<br/>


file 로  pod에 받기   

<br/>  

`/secrets` 폴더에 해당 화일이 있는지 확인한다.  

```bash
root@newedu:~# kubectl exec -it pod-secrets-via-file -- ls /secrets
password
```  

<br/>

환경변수로 받기  

pod에서 env로 확인한다.  

```bash
root@newedu:~# kubectl exec -it pod-secrets-via-env -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
TERM=xterm
HOSTNAME=pod-secrets-via-env
NSS_SDB_USE_CACHE=no
PASSWORD=secretpass
```  

<br/>


## 18. Ingress 구성 ( 배점 5점 이상 )


<br/>

> Question 1 :  Application Service 운영

> TASK :  
> `ingress-nginx` namespace 에  nginx 이미지를 `app=nginx` 레이블을 가지고 실행하는 `nginx` pod를 구성하세요.   
> 앞서 생성한 nginx Pod를 서비스 하는 `nginx` service를 생성하시오
> 현재 `appjs-service` 이름의 Service는 이미 동작중입니다. 별도 구성이 필요 없습니다. 

<br/>

> Question 2 :  Ingress 구성

> TASK :  
> `app-ingress.yaml` 화일을 생성하여  다움 조건의 ingress 서비스를 구성하시오.  
>  - ingress name : `app-ingress`  
>  - NODE_PORT:30080/ 접속 했을때 `nginx` 서비스로 연결 
>  - NODE_PORT:30080/app 접속 했을때 `appjs-service` 서비스로 연결 
>  - Ingress 구성에 다음의 annotations을 포함 시키시오.  
>     annotatons:
        kubernetes.io/ingress.class:nginx 

<br/>

OKD에 Ingress Controller 가 설치 되어 있지 않아 위에 구성중에 NODE_PORT는 서비스에 설정을 별도로 하고 테스트 하며 Namespace는 본인의 Namespace로 한다.  


<br/><br/>

>  결과값 

<br/>

30080 포트에 `/` 로 호출시 nginx에 접속 되어야 한다.  

```bash
root@newedu:~# curl 211.34.231.84:30080/
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```  

<br/>

30080 포트에 `/app` 로 호출시 nginx 오류가 발생.  

```bash
root@newedu:~# curl 211.34.231.84:30080/app
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```  

<br/>


## 19. Persistent Volume


<br/>

> Question  :  Create Persistent Volume

> TASK :  
> Create a persistent volume whith name `app-config` of capacity `1Gi` and access mode `ReadWriteMany`. 
> StorageClass: `az-c`
> The Type of volume is hostPath and its location is `/root/cka_pvc_test`.

<br/>

root폴더에서 아래 폴더를 생성한다.  

```bash
root@newedu:~# mkdir -p cka_pvc_test
```  


<br/><br/>

>  결과값 

<br/>

```bash
root@newedu:~# kubectl get pv app-config
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
app-config   1Gi        RWX            Recycle          Available           az-c                    10s
```  


<br/>


## 20. Persistent Volume Claim 을 사용하는 POD 운영 ( 자주나옴 )


<br/>

> Question  :  Application with PersistentVolumeClaim

> TASK 1 :  
> Create a new `PersistentVolumeClaim`   
> - Name : `app-volume`  
> - StorageClass : `az-c`  
> - Capacity : `10Mi`  

<br/>

> TASK 2 :  
> Create a new `Pod` which mounts the PersistentVolumeClaim as a volume   
> - Name : `web-server-pod`  
> - Image : `nginx`  
> - Mount path : `/usr/share/nginx/html`  

<br/>

> TASK 3 :  
> Configure the new Pod to have `ReadWriteMany` access on the volume.  

<br/>

참고 : https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/

<br/><br/>

>  결과값 

<br/>

```bash
root@newedu:~# kubectl get po web-server-pod
NAME             READY   STATUS    RESTARTS   AGE
web-server-pod   1/1     Running   0          8s
```  


<br/>


## 21. Check Resource Information


<br/>

> Question  : Check Resource Information
> TASK  :  
> List all `PV's sorted by name` saving the full `kubectl` output to `/my_volumes`.   
> Use `kubectl`'s own functionality for sorting the output, and do not manipulate it any further.   

<br/>

cheatsheet 에서 확인.  

https://kubernetes.io/docs/reference/kubectl/cheatsheet/  

JSON 으로 출력해서 가공한다.  

<br/><br/>

>  결과값

<br/>

```bash
root@newedu:~# cat my_volumes
```  
<br/>



## 23. Kubernetes TroubleShooting (1)

<br/>

> Question  : Not Ready 상태의 노드 활성화
> TASK  :  
> A kubernetes worker node, named `edu.worker01` in state `NotReady`.    
> Investigate why this is the case, and perform and appropriate steps to bring the node to a `Ready` state, ensuring that any changes are made permanent.



<br/><br/>

>  결과값

<br/>

아래 명령어를 실행해서 status를 확인한다.  

```bash
root@newedu:~# kubectl get nodes
```  
<br/>


## 24. User Role Binding

<br/>

Role은 특정 Namespace 에서만 적용. 

<br/>


> Question  : Configuring User API Authentification  

> TASK 1 :   Create the kubeconfig named `ckauser`.
> - username: `ckauser`    
> - certificate location: `/data/ca/ckauser.crt,/data/cka/ckauser.key`   
> - `ckauser` cluster must be operated with the privileges of the `ckauser` account.     

<br/>

> TASK 2 :   Create a role named `pod-role` that can `create,delete,watch,list,get` pods. Create the following rolebinding.   
> - name: `pod-rolebinding`    
> - role: `pod-role`   
> - user: `ckauser`     

<br/>

TASK 2 먼저 진행하고 2을 진행한다.  

아래 문서에서 `CSR` 로 검색하면 샘플이 나온다.  

<br/>

참고  
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/

<br/><br/>

>  결과값

<br/>

context를 변경하고 해당 유저로 pod를 생성 , 조회 해본다.     

```bash
root@newedu:~# kubectl config use-context ckauser
Switched to context "ckauser".
```  

<br/>


> 답안  

<br/>


아래 처럼 role 을 만들기 위한 yaml 화일을 생성한다.  ( 명령어로 진행 해도 됨 )

role_cka.yaml  
```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-role
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["create", "delete","watch", "list","get"]
```

<br/>

명령어 사용 하는 경우  

```bash
root@newedu:~# kubectl create role pod-role --verb=create --verb=get --verb=list --verb=watch --verb=delete --resource=pods
```
<br/>

role을 생성하고 확인한다.  

```bash
root@newedu:~# kubectl apply -f role_cka.yaml
role.rbac.authorization.k8s.io/pod-role created
root@newedu:~# kubectl get role pod-role
NAME       CREATED AT
pod-role   2022-09-27T14:04:40Z
```

<br/>

아래 처럼 rolebinding 을 만들기 위한 yaml 화일을 생성한다.  

rolebinding_cka.yaml    
```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-rolebinding
subjects:
- kind: User
  name: ckauser
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role 
  name: pod-role 
  apiGroup: rbac.authorization.k8s.io
```

<br/>

명령어를 사용하는 경우   

```bash
root@newedu:~# kubectl create rolebinding pod-rolebinding --role=pod-role --user=ckauser
```  

<br/>

rolebinding을 생성하고 확인한다.  

```bash
root@newedu:~# kubectl apply -f  rolebinding_cka.yaml
rolebinding.rbac.authorization.k8s.io/pod-rolebinding created
root@newedu:~# kubectl get rolebinding pod-rolebinding
NAME              ROLE            AGE
pod-rolebinding   Role/pod-role   24s
```

<br/>




## 25. User Cluster Role Binding

<br/>

ClusterRole은 Cluster 전체에 적용. 

<br/>


> Question  : Configuring User API Authentification  

> TASK  :  
>  Create a new `ClusterRole` named `app-clusterrole`, which only allows to `get,watch,list` the following resource types: `Deployment, Service`   
>  Bind the new ClusterRole `app-clusterrole` to the new  user `ckauser`.  
>  User ckauser and ckauser clusters are already configured.
>  To check the results, run the following command.
>   - kuectl config user-context ckauser


<br/><br/>

>  결과값

<br/>

<br/>

context를 변경하고  service 와 deployment를 조회해 본다.  

```bash
root@newedu:~# kubectl config use-context ckauser
root@newedu:~# kubectl get svc
root@newedu:~# kubectl get deploy
```  
  

<br/>


## 26. ServiceAccount Role Binding

<br/>

ServiceAccount는 pod의 권한이며 아무 것도 설정하지 않으면 default 가 적용된다.  

Role은 특정 Namespace 에서만 적용. 

<br/>


> Question  : Service Account , Role and RoleBinding  

> TASK  
> Create the `ServiceAccount` named pod-access in a new namespace called `edu29`.      
> Create a `Role` with the name `pod-role`, and the RoleBinding  named `pod-rolebinding`.      
> Map the ServiceAccount from the previous step to the API resources `Pods` with the operations `watch,list,get`.      

<br/>

여기에서는 reference를 사용합니다. 

<br/>

참고  : Docs -> reference로 이동하여 kubectl command line 선택  
- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

<br/><br/>

>  결과값

<br/>

아래 명령어 수행 후 확인.  

```bash
root@newedu:~# kubectl describe rolebinding pod-rolebinding -n edu29
```  

<br/>


## 27. ServiceAccount ClusterRole Binding ( SKIP )

<br/>

ServiceAccount는 pod의 권한이며 아무 것도 설정하지 않으면 default 가 적용된다.  

Role은 특정 Namespace 에서만 적용. 

<br/>


> Question  : Service Account , Role and RoleBinding  

> TASK  
> Create the `ServiceAccount` named pod-access in a new namespace called `edu29`.      
> Create a `Role` with the name `pod-role`, and the RoleBinding  named `pod-rolebinding`.      
> Map the ServiceAccount from the previous step to the API resources `Pods` with the operations `watch,list,get`.      

<br/>

여기에서는 reference를 사용합니다. 

<br/>

참고  : Docs -> reference로 이동하여 kubectl command line 선택  
- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

<br/><br/>

>  결과값

<br/>

아래 명령어 수행 후 확인.  

```bash
root@newedu:~# kubectl describe rolebinding pod-rolebinding -n edu29
```  

<br/>


## 28. Kube-DNS

<br/>

k8s 자체적으로 DNS 서버를 가지고 있다.

<br/>


> Question  : Service and DNS Lookup 구성  

> TASK  
> `Create` a `nginx` Pod called `nginx-resolver` using image `nginx` , expose it internally with a service  calls `nginx-resolver-service`.     
> `Test` that you are `able to look up the service and pod` names from within the cluster. Use the image `busybox:1.28` for dns lookup.     
> - Record results in `/tmp/nginx.svc` and `/tmp/nginx.pod`        
> - Pod : `nginx-resolver` created        
> - Service  `DNS Resolution` recorded correctly        
> - Pod `DNS resolution` recorded correctly        

<br/>

여기에서는 reference를 사용합니다. 

<br/>

참고  : Docs -> reference로 이동하여 kubectl command line 선택  
- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

<br/><br/>

>  결과값

<br/>

아래 명령어 수행 후 확인.  

```bash
root@newedu:~# cat /tmp/nginx.svc
Server:    172.30.0.10
Address 1: 172.30.0.10 dns-default.openshift-dns.svc.cluster.local

Name:      nginx-resolver-service.edu30.svc.cluster.local
Address 1: 172.30.127.12 nginx-resolver-service.edu30.svc.cluster.local
root@newedu:~# cat /tmp/nginx.pod
Server:    172.30.0.10
Address 1: 172.30.0.10 dns-default.openshift-dns.svc.cluster.local

Name:      10-131-7-82.edu30.pod.cluster.local
Address 1: 10.131.7.82 10-131-7-82.nginx-resolver-service.edu30.svc.cluster.local
```  

<br/>


## 29. Network Policy

<br/>

k8s 의 접근제어 라고 이해 하면 됨.

<br/>


> Question  : Network Ploicy with Namespace

> TASK  
> Create a New `NetworkPolicy` named `allow-port-from-namespace` in the existing namespace `devops`.       
> Ensure that the new NetworkPolicy `allows Pods in namespace edu30` to connect `port 80 of Pods in namespace devops`.      

<br/>

여기에서는 reference를 사용합니다. 

<br/>

참고  : Docs -> reference로 이동하여 kubectl command line 선택  
- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

<br/><br/>

>  결과값

<br/>

edu30 namespace 의 아무 pod에서 devops 네임스페이스의  webservice 서비스를₩ 80 포트로 호출한다.

<br/>
