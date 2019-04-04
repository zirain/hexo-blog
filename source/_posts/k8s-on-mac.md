---
title: k8s-on-mac
date: 2019-04-04 14:53:35
tags: 
    - docker
    - k8s
---

# Install minikube

使用阿里云修改的版本，避免国内无法连接 [k8s.gcr.io](https://k8s.gcr.io) 导致失败

`curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.0.0/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/`

# Install VM providers 

## Install Hyperkit driver

`brew install docker-machine-driver-hyperkit`

安装完成后需要执行如下命令：

docker-machine-driver-hyperkit need root owner and uid

`sudo chown root:wheel /usr/local/opt/docker-machine-driver-hyperkit/bin/docker-machine-driver-hyperkit`

`sudo chmod u+s /usr/local/opt/docker-machine-driver-hyperkit/bin/docker-machine-driver-hyperkit`

## Use VM driver

`minikube config set vm-driver hyperkit`

# Start minikube

`minikube start`

# Start minikube dashboard

`minikube dashboard`