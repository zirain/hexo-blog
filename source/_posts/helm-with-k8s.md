---
title: helm-with-k8s
date: 2019-04-22 17:25:30
tags:
    - helm
    - k8s
    - docker
---

# Install Helm

`helm version`

# Helm Init

`helm init --client-only --stable-repo-url https://aliacs-app-catalog.oss-cn-hangzhou.aliyuncs.com/charts/`
`helm repo add incubator https://aliacs-app-catalog.oss-cn-hangzhou.aliyuncs.com/charts-incubator/`
`helm repo update`
`helm repo list`

# Install Helm Tiller

`helm init --service-account tiller --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.12.3  --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts`

`helm init --service-account tiller --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.12.3 --tiller-tls-cert /etc/kubernetes/ssl/tiller001.pem --tiller-tls-key /etc/kubernetes/ssl/tiller001-key.pem --tls-ca-cert /etc/kubernetes/ssl/ca.pem --tiller-namespace kube-system --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts`

## 创建 Kubernetes 的服务帐号和绑定角色

`kubectl create serviceaccount --namespace kube-system tiller`
`kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller`
`kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}' deployment.extensions "tiller-deploy" patched`

## 查看是否授权成功
`kubectl get deploy --namespace kube-system   tiller-deploy  --output yaml|grep  serviceAccount`

## 验证 Tiller 是否安装成功

`kubectl -n kube-system get pods|grep tiller`
`helm version`

## 卸载 Helm 服务器端 Tiller

`helm reset`
`helm reset --force`