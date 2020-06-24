# Lab Setup

## Kubernetes stack information

- Ubuntu 20.04
- Microk8s (`snap install --classic --edge microk8s`)

### Configure Microk8s

#### Generate Kubectl configuration

```shell
$ cd $HOME
$ mkdir .kube
$ cd .kube
$ microk8s config > config
```

#### Get authentication token for dashboard

```shell
$ token=$(microk8s kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)

$ microk8s kubectl -n kube-system describe secret $token
Name:         default-token-x45zn
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: f4463baa-2638-4b54-b6e2-b803a1d755c9

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1103 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkJjSTU2YTB2WGpEeGlhSm12YlBCdXgzMGpGSmN2MEIwZlNCLVI5NFFPZ00ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkZWZhdWx0LXRva2VuLXg0NXpuIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImRlZmF1bHQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJmNDQ2M2JhYS0yNjM4LTRiNTQtYjZlMi1iODAzYTFkNzU1YzkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06ZGVmYXVsdCJ9.CzvHqwWfKwYfqGC6fCqhMCLn2aIIDEYnOjAYFrQzQB2DwinNYacX634Jg3R0OvUaEcMt6PCLVxAB_YxgzTHM9yzI7xYUDgKrlCmfAc1_fUd0Sp_3eoY3N34s3P5X_8CrqLheg7VVlOCMHkBsKeVg8nRG0hBz5jH4oKs-cySl6fA9KktUNIsMcJ3hN5aeHRsYJ6Rbt7YOWz_RLhpfKTtK9iWysINgk4FFEJ7b7yzSX0gTBYT1kW5KamTIVcaaO9MX-ClL4aEz4clrfZ11MlOaIJwD1M0P1e2SiDl4w_U4hX5qGGKujwHRi32xhE0kfxK42MORovi7u-VuegR7DcmX7A
```

### Configure plugins

### Activate plugins

#### DNS Plugin

```shell
$ microk8s enable dns

$ microk8s kubectl -n kube-system edit configmap/coredns
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health {
          lameduck 5s
        }
        ready
        log . {
          class error
        }
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . 8.8.8.8 8.8.4.4
        cache 30
        loop
        reload
        loadbalance
    }
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"Corefile":".:53 {\n    errors\n    health {\n      lameduck 5s\n    }\n    ready\n    log . {\n      class error\n    }\n    kubernetes cluster.local in-addr.arpa ip6.arpa {\n      pods insecure\n      fallthrough in-addr.arpa ip6.arpa\n    }\n    prometheus :9153\n    forward . 8.8.8.8 8.8.4.4\n    cache 30\n    loop\n    reload\n    loadbalance\n}\n"},"kind":"ConfigMap","metadata":{"annotations":{},"labels":{"addonmanager.kubernetes.io/mode":"EnsureExists","k8s-app":"kube-dns"},"name":"coredns","namespace":"kube-system"}}
```
