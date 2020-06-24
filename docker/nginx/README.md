![GitHub](https://img.shields.io/github/license/arista-netdevops-community/docker-avd-base) ![Docker Pulls](https://img.shields.io/docker/pulls/titom73/nginx) ![Docker Image Size (tag)](https://img.shields.io/docker/image-size/titom73/nginx/latest) ![Docker Image Version (tag latest semver)](https://img.shields.io/docker/v/titom73/nginx/latest)
# NGINX image

Small docker image running __`NGINX`__ on standard HTTP port and with a default page returning:

- Server Address
- Server name
- Server Date
- URI
- Request ID

__Docker image:__ [`titom73/nginx`](https://hub.docker.com/repository/docker/titom73/nginx)

## Example:

### Docker

```shell
➜ docker run --rm -d -p 32081:80 titom73/nginx
093b8ee991330560cdd1b6df47bc9d77510925e5caa9b28046316daf0d4f5b1b

➜ curl -s http://127.0.0.1:32081
Server address: 172.17.0.2:80
Server name: 093b8ee99133
Date: 24/Jun/2020:07:16:16 +0000
URI: /
Request ID: c06ffc69c6aae7e99efe0bbff235ca8d
```

### Kubernetes

```shell
$ kubectl apply -f https://raw.githubusercontent.com/titom73/k8s-journey/master/manifests/deploy-nginx-hello.yml
```
