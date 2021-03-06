---
title:  Docker(2)
date: 2019-01-26
tags: 
    - docker
---
# Docker

## 访问宿主机

### 方案一

宿主机执行ipconfig
会看到docker0那个ip，可以使用来访问宿主机

### 方案二

docker 18.03 加入了一个 feature，在容器中可以通过 host.docker.internal来访问主机
Use your internal IP address or connect to the special DNS name host.docker.internal which will resolve to the internal IP address used by the host.

## 查看IP

### 查看容器IP

`
{% raw %}
docker inspect --format '{{ .NetworkSettings.IPAddress }}' <container_id>
{% endraw %}
`

### 查看所有容器的IP

`
{% raw %}
docker inspect --format='{{.Name}} {{.State.Running}} {{range.NetworkSettings.Networks}} {{.IPAddress}}{{end}}' $(docker ps -aq)
{% endraw %}
`

## 删除容器

### 删除指定容器

`docker rm <container_id>`

`docker rm <container_name>`

`docker rm -f <container_id>`

`docker rm -f <container_name>`

### 删除所有容器

<code>docker rm &#96;docker ps -a -q&#96;</code>

## 删除镜像

`docker image rm <image_id>`

`docker rmi <image_id>`

### 删除所有镜像

<code>docker rmi &#96;docker images -a -q&#96;</code>