---
title:  Install Jenkins on Docker
date: 2018-12-04
tags: 
    - docker
    - jenkins
---
# Docker

## List containers

`docker ps`

## Stop container

`docker stop [docker-id]`

## Remove container

`docker rm [docker-id]`

## Run cmd in container

`docker exec -it [docker-name] [cmd] [options]`

## Install Jenkins

`dcoker run -d --name jenkins -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts`

## Install redis

`docker run -d --name my-redis -p 6379:6379 redis`

## Install rabbitmq

`docker run -d --name my-rabbit -p 5672:5672 -p 15672:15672 rabbitmq:3-management`

### Enable rabbitmq web management

`docker exec -it [docker-id] rabbitmq-plugins enable rabbitmq_management`