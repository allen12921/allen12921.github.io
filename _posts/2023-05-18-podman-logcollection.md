---
  title: "Collectioning Your Podman container log With filebeat"
categories:
  - Blog
tags:
  - filbeat
  - podman
  - logging
---
![filebeat](/assets/images/Filebeat.jpeg "filebeat")
# 什么是Filebeat?
 - Filebeat 是一款轻量级的日志收集工具，它属于 Elastic Stack（Elasticsearch、Logstash 和 Kibana）中的一个组件，主要用于监控和收集系统或应用程序的日志文件，并将这些日志数据发送到指定的Output进行进一步的处理和分析。
 - Filebeat 的工作原理是通过在需要收集日志的服务器上安装 Filebeat 代理，然后配置 Filebeat 以监控指定的日志文件或目录。当有新的日志数据生成时，Filebeat 会自动读取这些数据并将其发送到指定的 Kafka,Redis,Logstash,Elasticsearch 等Outputs。Filebeat 支持多种日志格式，如 Apache、Nginx、MySQL 等，并且具有自动处理日志轮转和多行事件等功能。
 - Filebeat 的优势在于它非常轻量级和高效，对系统资源的占用很低，同时具有高可扩展性和容错能力。这使得 Filebeat 成为在大型分布式环境中收集和传输日志数据的理想选择。和它同属Elastic-Beats家族的还有Auditbeat Functionbeat Metricbeat Heartbeat Packetbeat Winlogbeat Osquerybeat
# filebeat的配置
通常为了收集docker运行的container日志，我们需要使用到的filebeat配置主要分为三部分:
- inputs
```config
- type: container
  stream: stdout
  paths:
    - "/var/lib/docker/containers/*.log"
```
- processors
```config
processors:
  - add_docker_metadata:
      host: "unix:///var/run/docker.sock"
```
- outputs
```config
outputs:
  - stdout
```
# 如何使用Filebeat收集Podman运行的container日志
  由于filebeat默认是收集docker所运行的container日志，我们可以通过更改下面的配置来收集Podman运行的container日志:
