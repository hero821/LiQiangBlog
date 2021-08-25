---
title: 安装EFK
date: 2021-08-18 22:00:00
tags:
- K8S
- EFK
categories:
- [K8S]
---

# 介绍

EFK有两种组合方式，各有优缺点

elasticsearch + fluentd + kibana，是 kubernetes 官方推荐

elasticsearch + filebeat + kibana，是 elastic 技术栈

<!-- more -->

# 安装

## EFK | fluentd

### YAML 安装

参考：https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/fluentd-elasticsearch

### HELM 安装

```shell
# 添加仓库
helm repo add bitnami https://charts.bitnami.com/bitnami

# 部署 7.9.2 版本
helm search repo bitnami/elasticsearch --versions
helm pull bitnami/elasticsearch --version 12.8.0
tar zxvf elasticsearch-12.8.0.tgz 
cd elasticsearch
cp values.yaml values-override.yaml
vi values-override.yaml
###
master:
  replicas: 1
  heapSize: 256m
coordinating:
  replicas: 1
  heapSize: 256m
  service:
    type: NodePort
    nodePort: 30014
data:
  replicas: 1
  heapSize: 256m
###
helm install --create-namespace --namespace cmp-efk elasticsearch -f values-override.yaml .

# 部署 7.9.2 版本
helm search repo bitnami/kibana --versions
helm pull bitnami/kibana --version 5.3.14
tar zxvf kibana-5.3.14.tgz 
cd kibana
cp values.yaml values-override.yaml
vi values-override.yaml
###
service:
  type: NodePort
  nodePort: 30016
elasticsearch:
  hosts:
    - elasticsearch-elasticsearch-coordinating-only
  port: 9200
###
helm install --create-namespace --namespace cmp-efk kibana -f values-override.yaml .

# 部署 3.7.4 版本
helm search repo bitnami/fluentd --versions
helm pull bitnami/fluentd --version 3.7.4
tar zxvf fluentd-3.7.4.tgz 
cd fluentd
cp values.yaml values-override.yaml
vi values-override.yaml
###
forwarder:
  configMapFiles:
    fluentd-inputs.conf: |
      <source>
        @type http
        port 9880
      </source>
      <source>
        @id fluentd-containers.log
        @type tail
        path /var/log/containers/*.log
        pos_file /opt/bitnami/fluentd/logs/buffers/fluentd-docker.pos
        tag raw.kubernetes.*
        read_from_head true
        <parse>
          @type multi_format
          <pattern>
            format json
            time_key time
            time_format %Y-%m-%dT%H:%M:%S.%NZ
          </pattern>
          <pattern>
            format /^(?<time>.+) (?<stream>stdout|stderr) [^ ]* (?<log>.*)$/
            time_format %Y-%m-%dT%H:%M:%S.%N%:z
          </pattern>
        </parse>
      </source>
      <match raw.kubernetes.**>
        @id raw.kubernetes
        @type detect_exceptions
        remove_tag_prefix raw
        message log
        stream stream
        multiline_flush_interval 5
        max_bytes 500000
        max_lines 1000
      </match>
      <filter **>
        @id filter_concat
        @type concat
        key message
        multiline_end_regexp /\n$/
        separator ""
      </filter>
      <filter kubernetes.**>
        @id filter_kubernetes_metadata
        @type kubernetes_metadata
      </filter>
      <filter kubernetes.**>
        @id filter_parser
        @type parser
        key_name log
        reserve_data true
        remove_key_name_field true
        <parse>
          @type multi_format
          <pattern>
            format json
          </pattern>
          <pattern>
            format none
          </pattern>
        </parse>
      </filter>
      <filter kubernetes.**>
        @id filter_record_transformer_kubernetes
        @type record_transformer
        enable_ruby true
        <record>
          level INFO
          _tmp1_ ${record["kubernetes"]["namespace"] = record["kubernetes"]["namespace_name"]}
          _tmp2_ ${record["kubernetes"]["pod"] = {"name": record["kubernetes"]["pod_name"]}}
          _tmp3_ ${record["kubernetes"]["container"] = {"name": record["kubernetes"]["container_name"]}}
          _tmp4_ ${record["kubernetes"]["node"] = {"name": record["kubernetes"]["host"]}}
        </record>
        remove_keys _tmp1_,_tmp2_,_tmp3_,_tmp4_
      </filter>
aggregator:
  configMapFiles:
    fluentd-output.conf: |
      <match fluentd.healthcheck>
        @type stdout
      </match>
      <match **>
        @id elasticsearch
        @type elasticsearch
        @log_level info
        type_name _doc
        include_tag_key true
        host "#{ENV['ELASTICSEARCH_HOST']}"
        port "#{ENV['ELASTICSEARCH_PORT']}"
        logstash_format true
        <buffer>
          @type file
          path /opt/bitnami/fluentd/logs/buffers/logs.buffer
          flush_mode interval
          retry_type exponential_backoff
          flush_thread_count 2
          flush_interval 5s
          retry_forever
          retry_max_interval 30
          chunk_limit_size 2M
          total_limit_size 500M
          overflow_action block
        </buffer>
      </match>
  extraEnv:
    - name: ELASTICSEARCH_HOST
      value: "elasticsearch-elasticsearch-coordinating-only"
    - name: ELASTICSEARCH_PORT
      value: "9200"
###
helm install --create-namespace --namespace cmp-efk fluentd -f values-override.yaml .
```

## EFK | filebeat

### HELM 安装

```shell
helm repo add elastic https://helm.elastic.co

helm pull elastic/elasticsearch --version 7.9.2
helm pull elastic/kibana --version 7.9.2
helm pull elastic/filebeat --version 7.9.2

vi elasticsearch-values.yaml
###
fullnameOverride: elasticsearch
replicas: 1
minimumMasterNodes: 1
service:
  type: NodePort
  nodePort: 30014
###
vi filebeat-values.yaml
###
fullnameOverride: filebeat
###
vi kibana-values.yaml
###
fullnameOverride: kibana
service:
  type: NodePort
  nodePort: 30016
###

helm install --create-namespace --namespace cmp-efk elasticsearch -f elasticsearch-values.yaml ./elasticsearch
helm install --create-namespace --namespace cmp-efk filebeat -f filebeat-values.yaml ./filebeat
helm install --create-namespace --namespace cmp-efk kibana -f kibana-values.yaml ./kibana
```

