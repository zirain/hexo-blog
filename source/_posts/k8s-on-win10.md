---
title: Install Kubernetes on Windows 10
date: 2019-01-27
tags: 
    - docker
    - k8s
---
## Install Minikube

### 准备条件

- chocolate
- VPN

### 安装minikube

`choco install minikube kubernetes-cli`

国内无法连接 [k8s.gcr.io](https://k8s.gcr.io) , 可以使用aliyun提供的minikube来替代，下载 [minikube-windows-amd64.exe](http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.4.2/minikube-windows-amd64.exe) 文件，重命名为 `minikube.exe`，并设置环境变量

`minikube start --registry-mirror=https://registry.docker-cn.com --kubernetes-version v1.13.2`

### 设置hyper-v switch

### 启动minikube

`minikube start --registry-mirror=https://registry.docker-cn.com --vm-driver="hyperv" --hyperv-virtual-switch="minikubeSwitch"`

### 错误处理

如果你遇到这个错误，Error restarting cluster: restarting kube-proxy: waiting for kube-proxy to be up for configmap update: timed out waiting for the condition`

通过 `minikube delete`，`minikube start` 可以解决

### 启动dashboard

`minikube dashboard`