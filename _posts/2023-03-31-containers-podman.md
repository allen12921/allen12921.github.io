---
title: "Manage Your Containers With Podman"
categories:
  - Blog
tags:
  - containers
  - podman
  - rootless
---
![ClickHouse](/assets/images/podman.svg "podman")
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
  - 运行容器
  - 运行pod

# Podman与Docker的比较
  - Podman不需要守护进程，而Docker需要。
  - Podman可以以Pod的形式运行容器，而Docker没有这个功能。
  - Podman默认可运行rootless容器，而Docker需要特别安装设置[^2]。
  - Podman与Docker都遵循OCI规范。
  - Podman不支持Amazon Linux2,而Docker支持


[^1]: PodMan Can Creates pods or volumes based on the Kubernetes kind described in the YAML. Supported kinds are Pods, Deployments and PersistentVolumeClaims.
[^2]: https://docs.docker.com/engine/security/rootless/