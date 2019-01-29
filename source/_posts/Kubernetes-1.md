---
title: Kubernetes(1)
date: 2019-01-27
tags: 
    - docker
    - k8s
---
# Install Kubernetes on Windows 10

## Install Minikube

### 准备条件

- chocolate
- VPN

### 安装minikube

`choco install minikuber kubernetes-cli`

`minikube start --registry-mirror=https://registry.docker-cn.com --kubernetes-version v1.13.2`

### 设置hyper-v switch

### 启动minikube

`minikube start --vm-driver="hyperv" --hyperv-virtual-switch="minikubeSwitch"`

### 错误处理

如果你遇到这个错误，Error restarting cluster: restarting kube-proxy: waiting for kube-proxy to be up for configmap update: timed out waiting for the condition`

通过 `minikube delete`，`minikube start` 可以解决

遇到无法连接 k8s.gcr.io错误时:可以通过镜像拉取

`docker pull mirrorgooglecontainers/kube-apiserver-amd64:v1.11.7`

`docker pull mirrorgooglecontainers/kube-controller-manager-amd64:v1.11.7`

`docker pull mirrorgooglecontainers/kube-scheduler-amd64:v1.11.7`

`docker pull mirrorgooglecontainers/kube-proxy-amd64:v1.11.7`

`docker pull mirrorgooglecontainers/pause:3.1`

`docker pull mirrorgooglecontainers/etcd-amd64:3.2.24`

`docker pull coredns/coredns:1.2.6`

通过docker tag命令来修改镜像的标签:

`docker tag docker.io/mirrorgooglecontainers/kube-proxy-amd64:v1.11.7 k8s.gcr.io/kube-proxy-amd64:v1.13.2`

`docker tag docker.io/mirrorgooglecontainers/kube-scheduler-amd64:v1.11.7 k8s.gcr.io/kube-scheduler-amd64:v1.13.2`

`docker tag docker.io/mirrorgooglecontainers/kube-apiserver-amd64:v1.11.7 k8s.gcr.io/kube-apiserver-amd64:v1.13.2`

`docker tag docker.io/mirrorgooglecontainers/kube-controller-manager-amd64:v1.11.7 k8s.gcr.io/kube-controller-manager-amd64:v1.13.2`

`docker tag docker.io/mirrorgooglecontainers/etcd-amd64:3.2.24  k8s.gcr.io/etcd-amd64:3.2.24`

`docker tag docker.io/mirrorgooglecontainers/pause:3.1  k8s.gcr.io/pause:3.1`

`docker tag docker.io/coredns/coredns:1.2.6  k8s.gcr.io/coredns:1.2.6`
