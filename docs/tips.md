# Tips & Tricks

- Get a shell in namespace:

```shell
$ kubectl run --restart=Never --rm -it --image=alpine alpine-runner
```

- Set alias to start shell in k8s

```shell
alias kalpine='kubectl run --restart=Never --rm -it --image=alpine alpine-runner'
```

- Install [kns](https://github.com/blendle/kns)


```shell
$ brew tap blendle/blendle
$ brew install kns
```