---
title: "Manage Your Containers With Podman"
categories:
  - Blog
tags:
  - containers
  - podman
  - rootless
---
![Posman](/assets/images/podman.svg "podman")
# 什么是Podman?
  - Podman是一个基于容器的开源工具，可以用于管理和运行OCI（Open Container Initiative）容器。
  - Podman与Docker类似，但不需要守护进程，也不需要root权限来运行容器。
  - Podman可以在Linux、macOS和Windows上运行。

# Podman的特点
  - Podman可以在没有守护进程的情况下运行容器，这意味着更安全、更灵活。
  - Podman可以运行rootless容器，这意味着不需要root权限即可运行容器。
  - Podman可以以Pod的形式运行容器，这意味着可以方便地管理多个容器。
  - Podman可以生成和运行适用于Kubernetes的YAML[^1]
  - Podman完全兼容Docker CLI，因此可以直接通过alias docker=podman来平滑切换docker到podman。


# 如何使用Podman
  - 安装
  
    - MacOS: 
      ```bash
       brew install podman&&podman machine init&&podman machine start
      ```
    - Windows
       Get installer from https://github.com/containers/podman/release and install it[^3]
       ```batch
         PS C:\Users\User> podman machine init
       ```
    - Linux
      预编译包支持的Linux版本包括Arch Linux,Manjaro Linux,Alpine Linux,CentOS7+,Debian 11+,Fedora 35+,Ubuntu 20.10+,使用系统内的包管理工具安装即可。
      当然也可从源码安装，源码安装如果不是官方支持的版本，则需要自行处理依赖问题。
  - 配置文件
    - registries.conf: 此文件配置了当未指定镜像仓库地址时，需要去查询的镜像仓库地址。
    - mounts.conf: 此文件配置了运行podman run和build命令时，自动复制到container中的内容，它不会被提交到最终的image中，常用来挂载第三方密钥信息。
    - seccomp.json: 此文件包含了容器中运行执行的seccomp rules。
    - policy.json: 此文件用于设置是否接受image和验证签名是否有效的规则。
  - 运行容器
    ```bash
       podman run -it docker.io/library/busybox 
    ```
  - 运行pod
    ```bash
       podman pod create --name mypod --publish 80:80 --publish 3306:3306
       podman container create --pod mypod --name nginx  nginx
       podman container create --pod mypod --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=tru mysql
       podman pod start mypod
    ```
# Podman与Docker的比较
  - Podman不需要守护进程，而Docker需要。
  - Podman可以以Pod的形式运行容器，而Docker没有这个功能。
  - Podman的镜像是按用户级别进行隔离存储的，而Docker的镜像是所有用户共用的。
  - Podman默认可运行rootless容器，而Docker需要特别安装设置[^2]。
  - Podman与Docker都遵循OCI规范。
  - Podman不支持Amazon Linux2,而Docker支持。


[^1]: PodMan Can Creates pods or volumes based on the Kubernetes kind described in the YAML. Supported kinds are Pods, Deployments and PersistentVolumeClaims.
[^2]: https://docs.docker.com/engine/security/rootless/
[^3]: https://github.com/containers/podman/blob/main/docs/tutorials/podman-for-windows.md
