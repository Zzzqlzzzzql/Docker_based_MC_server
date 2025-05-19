# Docker 和 Kubernetes Minecraft 服务器部署项目 

本项目提供了一系列使用 Docker 和 Kubernetes 来部署 Minecraft Java 版服务器 (PaperMC) 及 BungeeCord 代理的配置和指南。旨在帮助用户快速搭建、管理和扩展自己的 Minecraft 服务器环境。

## 项目概览

本项目探索并实现了多种 Minecraft 服务器的部署方案，包括：

* **基于 Docker 的独立服务器部署**：为单个 PaperMC 服务器或 BungeeCord 代理提供 Dockerfile 和运行指南。
* **基于 Kubernetes 的集群部署**：提供一套完整的 YAML 配置文件，用于在 Kubernetes 集群上部署一个包含 BungeeCord 代理和多个后端 PaperMC 服务器（例如，一个 "lobby" 服务器和一个 "new" 服务器）的高可用、可伸缩的 Minecraft 服务器网络。
* **服务器管理面板 (MCSManager)**：集成了关于使用 MCSManager 进行服务器管理的信息。
* **嵌入式部署探索**：包含了将 Minecraft 服务器部署到嵌入式设备上的相关思路。
* **性能优化和集群信息**：提供了关于服务器性能优化和使用 Docker Swarm/Kubernetes 进行集群管理的参考信息。

## 主要特点

* 使用 PaperMC 作为 Minecraft 服务端核心，性能优越。
* 通过 BungeeCord 实现多服务器连接与跳转。
* 提供 Dockerfile 用于构建自定义的 Minecraft 和 BungeeCord 镜像。
* 提供 Kubernetes YAML 清单文件，实现声明式部署和管理。
* 支持持久化存储，确保游戏数据在 Pod 重启后不丢失 (Kubernetes 部分)。
* 易于扩展，可以根据需求增加更多的后端 Minecraft 服务器实例。

## 仓库结构

```
.
├── MC_docker/
│   ├── 1_21/
│   │   ├── Dockerfile
│   │   ├── server-start.sh
│   │   ├── spigot.yml
│   │   └── README.md
│   ├── 1_21base/
│   │   ├── Dockerfile
│   │   ├── server.properties
│   │   ├── spigot.yml
│   │   └── README.md
│   ├── bungeecord/
│   │   ├── Dockerfile
│   │   ├── config.yml
│   │   └── README.md
│   └── optimized/
│       └── info.md
├── k8s/
│   ├── 00-namespace.yaml
│   ├── 01-mc-server-statefulset.yaml
│   ├── 02-mc-server-headless-service.yaml
│   ├── 03-mc-new-server-statefulset.yaml
│   ├── 04-mc-new-headless-service.yaml
│   ├── 05-bungeecord-configmap.yaml
│   ├── 06-bungeecord-deployment.yaml
│   ├── 07-bungeecord-service.yaml
│   └── README.md
├── MCSmanager/
│   └── info.md
├── embedded/
├── 总结.md
└── README.md
```

## 部署选项与指南

本项目提供了多种部署 Minecraft 服务器的方法。请根据您的需求选择合适的方案，并参考对应目录下的详细 README 文件。

### 1. 使用 Docker 单独部署

如果您希望在单个主机上快速运行 Minecraft 服务器或 BungeeCord 代理，可以参考 `MC_docker/` 目录下的配置。

* **部署 PaperMC 1.21 服务器**:
    * 请参考: [`MC_docker/1_21/README.md`](MC_docker/1_21/README.md)
* **构建基础 PaperMC 1.21 镜像**:
    * 请参考: [`MC_docker/1_21base/README.md`](MC_docker/1_21base/README.md)
* **部署 BungeeCord 代理服务器**:
    * 请参考: [`MC_docker/bungeecord/README.md`](MC_docker/bungeecord/README.md)

### 2. 使用 Kubernetes 部署集群

如果您希望部署一个可伸缩、高可用的 Minecraft 服务器集群（包含 BungeeCord 代理和多个后端 PaperMC 服务器），请参考 `k8s/` 目录下的配置和指南。

* **详细部署步骤、配置说明和管理命令**:
    * 请参考: [`k8s/README.md`](k8s/README.md) 

### 3. 使用 MCSManager 管理面板

MCSManager 是一个流行的游戏服务器管理面板，本项目提供了相关信息。

* **了解 MCSManager 的功能和技术特点**:
    * 请参考: [`MCSmanager/info.md`](MCSmanager/info.md)

### 4. 嵌入式部署探索

本项目还初步探讨了将 Minecraft 服务器部署到嵌入式设备上的可能性。

* **相关思路和要求**:
    * 请参考: [`embedded/info.md`](embedded/info.md)

## 通用先决条件

* **Docker**: 对于所有基于 Docker 的部署方案，您都需要安装 Docker Engine。
* **Kubernetes 集群**: 对于 Kubernetes 部署方案，您需要一个可用的 Kubernetes 集群和 `kubectl` 命令行工具。
* **Minecraft 知识**: 对 Minecraft 服务器和 BungeeCord 的基本配置和管理有一定的了解。

## 贡献

欢迎对本项目进行贡献！如果您有任何改进建议或发现了问题，请随时创建 Issue 或提交 Pull Request。