---
title:  Docker(1)
date: 2019-01-23
tags: 
    - docker
    - elk
---
# Docker

## Install Elasticsearch & Kibana  

### create network

`docker network create es-network`

### install elasticsearch

`docker run -d --name elasticsearch --net es-network -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.5.4`

### install kibana

`docker run -d --name kibana --net es-network -p 5601:5601 docker.elastic.co/kibana/kibana:6.5.4`

## Run consul in devlopment mode

`docker run --name consul1 -p 8500:8500 consul`