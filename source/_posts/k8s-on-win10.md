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

国内无法连接 [k8s.gcr.io](https://k8s.gcr.io) , 可以使用aliyun提供的minikube来替代，下载 [minikube-windows-amd64.exe](https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.12.1/minikube-windows-amd64.exe?spm=a2c6h.12873639.0.0.ab202043XcPh8G&file=minikube-windows-amd64.exe) 文件，重命名为 `minikube.exe`，并设置环境变量

[参考资料](https://developer.aliyun.com/article/221687)

### 设置hyper-v switch

### 启动minikube

```
minikube start --registry-mirror=https://hub-mirror.c.163.com --vm-driver="hyperv" --hyperv-virtual-switch="minikubeSwitch" --image-mirror-country cn --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.12.0.iso --memory=4096
```

### 错误处理

如果你遇到这个错误，Error restarting cluster: restarting kube-proxy: waiting for kube-proxy to be up for configmap update: timed out waiting for the condition`

通过 `minikube delete`，`minikube start` 可以解决

### 启动dashboard

`minikube dashboard`