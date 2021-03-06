# NGINX with Ingress based on vhost

## Description

Deploy an NGINX pod and allow access from outside using traefik as ingress controller using virtual host.

## Deploy content

Complete manifest is available under [__manifests__](https://github.com/titom73/k8s-journey/blob/master/manifests/deploy-nginx-basic.yml) folder

### Create deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-basics
spec:
  selector:
    matchLabels:
      app: webui
  replicas: 1
  template:
    metadata:
      labels:
        app: webui
    spec:
      containers:
      - name: nginx
        image: titom73/nginx
        ports:
        - containerPort: 80
```

### Expose service

Service is only available with `ClusterIP` and could not be reached from outside.

```yaml
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx-basics
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: webui
  type: ClusterIP
```

### Create Ingress

```yaml
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: nginx-basics
spec:
  rules:
  - host: test.lab.as73.inetsix.net
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx-basics
          servicePort: 80
```

Host __MUST__ be changed according your own setup. [nip.io](https://nip.io) might provide a solution to create your host

### Use built-in manifest

```shell
$ kubectl apply -f manifest/deploy-nginx-basic.yml
deployment.apps/nginx-basics created
service/nginx-basic created
ingress.networking.k8s.io/nginx-basics created
```

## Check result

### Traefik validation

![traefi$ kubectl dashboard](medias/traefik-dashboard.png)

### From shell

- Shell in Kubernetes using SVC name

```shell
$ kalpine
If you don't see a command prompt, try pressing enter.
/ # ap$ kubectl add curl
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
(1/4) Installing ca-certificates (20191127-r4)
(2/4) Installing nghttp2-libs (1.41.0-r0)
(3/4) Installing libcurl (7.69.1-r0)
(4/4) Installing curl (7.69.1-r0)
Executing busybox-1.31.1-r16.trigger
Executing ca-certificates-20191127-r4.trigger
OK: 7 MiB in 18 packages

/ # curl nginx-basics

Server address: 10.1.32.32:80
Server name: nginx-basics-c48d789fd-kzcbt
Date: 23/Jun/2020:13:46:42 +0000
URI: /
Request ID: 36ee8606b2ecb63d877f7ebc8c49d3a5
```

- Shell in Kubernetes using SVC name

```shell
/ # curl test.lab.as73.inetsix.net/basics
Server address: 10.1.32.32:80
Server name: nginx-basics-c48d789fd-kzcbt
Date: 23/Jun/2020:13:56:30 +0000
URI: /basics
Request ID: e1b7e84d56315e41cd3864ae37b921c0
```

- Shell from laptop

```shell
@tomcat ➜ ~  curl test.lab.as73.inetsix.net/basics
Server address: 10.1.32.32:80
Server name: nginx-basics-c48d789fd-kzcbt
Date: 23/Jun/2020:13:52:37 +0000
URI: /basics
Request ID: 9d2133fc127ab172cf3dfc0a95f6554d
```

## Chek Kubernetes status

### Chek deployment

```shell
$ kubectl get deployments
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
nginx-basics        1/1     1            1           28m
```

And get more details:

```
$ kubectl describe deployments.apps nginx-basics
Name:                   nginx-basics
Namespace:              default
CreationTimestamp:      Tue, 23 Jun 2020 12:18:05 +0200
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
                        kubectl.kubernetes.io/last-applied-configuration:
                          {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"nginx-basics","namespace":"default"},"spec":{"replicas":1...
Selector:               app=webui
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=webui
  Containers:
   nginx:
    Image:        nginx
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-basics-8c54c7598 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  30m   deployment-controller  Scaled up replica set nginx-basics-8c54c7598 to 1
  Normal  ScalingReplicaSet  29m   deployment-controller  Scaled up replica set nginx-basics-8c54c7598 to 3
  Normal  ScalingReplicaSet  29m   deployment-controller  Scaled down replica set nginx-basics-8c54c7598 to 1
```

### Check POD

```shell
$ kubectl get pod
NAME                                 READY   STATUS    RESTARTS   AGE
nginx-basics-8c54c7598-rd475         1/1     Running   0          34m
```

```shell
Name:         nginx-basics-8c54c7598-rd475
Namespace:    default
Priority:     0
Node:         k8s-node1/10.73.1.248
Start Time:   Tue, 23 Jun 2020 12:18:05 +0200
Labels:       app=webui
              pod-template-hash=8c54c7598
Annotations:  <none>
Status:       Running
IP:           10.1.32.28
IPs:
  IP:           10.1.32.28
Controlled By:  ReplicaSet/nginx-basics-8c54c7598
Containers:
  nginx:
    Container ID:   containerd://c738ac9d6c8d7ee00e5d56e7d422f7873c5333c3df70528367989623dbc51343
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:21f32f6c08406306d822a0e6e8b7dc81f53f336570e852e25fbe1e3e3d0d0133
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 23 Jun 2020 12:18:07 +0200
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-tp69t (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-tp69t:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-tp69t
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From                Message
  ----    ------     ----  ----                -------
  Normal  Scheduled  35m   default-scheduler   Successfully assigned default/nginx-basics-8c54c7598-rd475 to k8s-node1
  Normal  Pulling    35m   kubelet, k8s-node1  Pulling image "nginx"
  Normal  Pulled     35m   kubelet, k8s-node1  Successfully pulled image "nginx"
  Normal  Created    35m   kubelet, k8s-node1  Created container nginx
  Normal  Started    34m   kubelet, k8s-node1  Started container nginx
```

### Check Service

```shell
$ kubectl get services
NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes          ClusterIP   10.152.183.1     <none>        443/TCP        144m
nginx-basics        ClusterIP   10.152.183.49    <none>        80/TCP         38m
nginx-hello-world   NodePort    10.152.183.104   <none>        80:31788/TCP   41m
```

Display service details

```shell
$ kubectl describe service nginx-basics
Name:              nginx-basics
Namespace:         default
Labels:            app=nginx
Annotations:       kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"nginx"},"name":"nginx-basics","namespace":"default"},"sp...
Selector:          app=webui
Type:              ClusterIP
IP:                10.152.183.49
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.1.32.28:80
Session Affinity:  None
Events:            <none>
```

### Check ingress services

```shell
$ kubectl get ingress
NAME           CLASS    HOSTS                       ADDRESS   PORTS   AGE
nginx-basics   <none>   test.lab.as73.inetsix.net             80      40m
```

```shell
$ kubectl describe ingress nginx-basics
Name:             nginx-basics
Namespace:        default
Address:
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host                       Path  Backends
  ----                       ----  --------
  test.lab.as73.inetsix.net
                             /   nginx-basics:80 (10.1.32.28:80)
Annotations:
  kubectl.kubernetes.io/last-applied-configuration:  {"apiVersion":"networking.k8s.io/v1beta1","kind":"Ingress","metadata":{"annotations":{},"name":"nginx-basics","namespace":"default"},"spec":{"rules":[{"host":"test.lab.as73.inetsix.net","http":{"paths":[{"backend":{"serviceName":"nginx-basics","servicePort":80},"path":"/"}]}}]}}

Events:  <none>
```

## Next steps

### Specific load balancing

By default, ingress load balance traffic using round robin approach on all replicas of a POD deployed matching `app=nginx-basics`. This part needs to deploy 3 different PODs to load balance across 3 different PODs

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: cheeseplate
  annotations:
    traefik.ingress.kubernetes.io/service-weights: |
      basics01: 50%
      basics02: 25%
      basics03: 25%
spec:
  rules:
  - host: test2.lab.as73.inetsix.net
    http:
      paths:
      - path: /
        backend:
          serviceName: basics01
          servicePort: 80
      - path: /
        backend:
          serviceName: basics02
          servicePort: 80
      - path: /
        backend:
          serviceName: basics03
          servicePort: 80
```
