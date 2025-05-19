# Kubernetes 部署 Minecraft 服务器集群 (BungeeCord + 双 PaperMC 后端)

本项目包含一套 Kubernetes YAML 配置文件，用于部署一个由 BungeeCord 代理的 Minecraft 服务器集群。该集群包含两个服务器实例：一个作为 "lobby" (大厅/主城)，另一个作为 "new" (其他游戏模式)。

## 目录结构 (假设所有文件在同一目录)

- `00-namespace.yaml`: 定义 Kubernetes 命名空间。
- `01-mc-paper-statefulset.yaml`: 定义 "lobby" 服务器的 StatefulSet。
- `02-mc-paper-headless-service.yaml`: 为 "lobby" 服务器定义 Headless Service。
- `03-mc-new-server-statefulset.yaml`: 定义 "new" 服务器的 StatefulSet。
- `04-mc-new-server-headless-service.yaml`: 为 "new" 服务器定义 Headless Service。
- `05-bungeecord-configmap.yaml`: 包含 BungeeCord 的 `config.yml` 配置文件。
- `06-bungeecord-deployment.yaml`: 定义 BungeeCord 代理服务器的 Deployment。
- `07-bungeecord-service.yaml`: 为 BungeeCord 代理定义 LoadBalancer Service，以便外部访问。

## 先决条件

1.  **Kubernetes 集群**: 需要一个正在运行的 Kubernetes 集群。例如，可以使用 Docker Desktop 内置的 Kubernetes, Minikube, Kind, 或者云提供商的 Kubernetes 服务。
2.  **`kubectl` 命令行工具**: 用于与 Kubernetes 集群进行交互。
3.  **Docker 镜像**:
    * **PaperMC 镜像**: 需要为 PaperMC 服务器构建一个或多个 Docker 镜像。
        * 在 `01-mc-paper-statefulset.yaml` 中，`lobby` 服务器使用的镜像为 `server:latest`，需要借助 `./MC_docker/1_21/Dockerfile` 生成
        * 在 `03-mc-new-server-statefulset.yaml` 中，`new` 服务器使用的镜像需要内置 `new-server:latest`，需要借助 `./MC_docker/1_21base/Dockerfile` 生成
        * 这两个服务器可以使用相同的基础镜像，但其启动端口和可能的其他配置（如世界名称）需要在镜像层面或通过启动脚本/环境变量（如果镜像支持）进行区分。
    * **BungeeCord 镜像**: 需要一个 BungeeCord 的 Docker 镜像。在 `06-bungeecord-deployment.yaml` 中，`bungeecord:latest` 需要指向这个镜像。

## 配置说明

在部署之前，可能需要检查并修改以下 YAML 文件中的一些关键配置：

* **`01-mc-server-statefulset.yaml` (Lobby 服务器)**:
    * `spec.template.spec.containers[0].image`: 替换为您为 `lobby` 服务器构建的镜像名称和标签。
    * `spec.template.spec.containers[0].ports[0].containerPort`: 应为 `25535`。
    * `resources`: 根据需要调整 CPU 和内存请求与限制。
    * `volumeClaimTemplates.spec.resources.requests.storage`: 调整世界数据的存储大小。
* **`02-minecraft-headless-service.yaml` (Lobby 服务)**:
    * `metadata.name`: 应与 `01-minecraft-statefulset.yaml` 中的 `spec.serviceName` 匹配。
    * `spec.ports[0].port` 和 `targetPort`: 应为 `25535`。
* **`03-mc-new-server-statefulset.yaml` (New 服务器)**:
    * `spec.template.spec.containers[0].image`: 替换为您为 `new` 服务器构建的镜像名称和标签。
    * `spec.template.spec.containers[0].ports[0].containerPort`: 应为 `25565`。
    * `resources`: 根据需要调整。
    * `volumeClaimTemplates.spec.resources.requests.storage`: 调整存储大小。
* **`04-mc-new-headless-service.yaml` (New 服务)**:
    * `metadata.name`: 应与 `06-mc-new-server-statefulset.yaml` 中的 `spec.serviceName` 匹配。
    * `spec.ports[0].port` 和 `targetPort`: 应为 `25565`。
* **`05-bungeecord-configmap.yaml` (BungeeCord 配置)**:
    * `data.config.yml`:
        * `servers.lobby.address`: 应为 `mc-paper-0.<lobby服务器的Headless Service名称>.minecraft-server.svc.cluster.local:25535` (请确保服务名称与 `02-minecraft-headless-service.yaml` 中的 `metadata.name` 匹配)。
        * `servers.new.address`: 应为 `mc-new-server-0.<new服务器的Headless Service名称>.minecraft-server.svc.cluster.local:25565` (请确保服务名称与 `07-mc-new-headless-service.yaml` 中的 `metadata.name` 匹配)。
        * 其他 BungeeCord 相关设置，如 MOTD, `max_players`, `ip_forward` 等。
* **`06-bungeecord-deployment.yaml` (BungeeCord 部署)**:
    * `spec.template.spec.containers[0].image`: 替换为您的 BungeeCord 镜像名称和标签。
    * `resources`: 根据需要调整 CPU 和内存。

## 部署步骤

1.  **创建命名空间**:
    ```bash
    kubectl apply -f 00-namespace.yaml
    ```

2.  **部署所有其他资源**:
    建议按顺序或按依赖关系应用，但如果所有文件都在一个目录且内部命名正确，可以一起应用：
    ```bash
    # 部署 Lobby 服务器
    kubectl apply -f 01-mc-server-statefulset.yaml -n minecraft-server
    kubectl apply -f 02-mc-server-headless-service.yaml -n minecraft-server

    # 部署 New 服务器
    kubectl apply -f 06-mc-new-server-statefulset.yaml -n minecraft-server
    kubectl apply -f 07-mc-new-headless-service.yaml -n minecraft-server

    # 部署 BungeeCord
    kubectl apply -f 03-bungeecord-configmap.yaml -n minecraft-server
    kubectl apply -f 04-bungeecord-deployment.yaml -n minecraft-server
    kubectl apply -f 05-bungeecord-service.yaml -n minecraft-server
    ```
    或者，如果所有文件都在当前目录：
    ```bash
    kubectl apply -f . -n minecraft-server
    ```

3.  **（可选）重启 BungeeCord Pod**: 如果您在部署 BungeeCord 后修改了 `05-bungeecord-configmap.yaml`，您需要重启 BungeeCord Pod 来加载新的配置：
    ```bash
    kubectl rollout restart deployment bungeecord-proxy -n minecraft-server
    ```

## 访问服务器

1.  **获取 BungeeCord 外部 IP 和端口**:
    ```bash
    kubectl get service bungeecord-proxy-external -n minecraft-server
    ```
    查找 `EXTERNAL-IP` (在 Docker Desktop 上通常是 `localhost`) 和 `PORT(S)` (应该是 `25577`)。

2.  **连接 Minecraft 客户端**:
    * 服务器地址: `localhost:25577` (或获取到的 `EXTERNAL-IP` 和端口)。

3.  **在游戏内切换服务器**:
    * 默认您会进入 BungeeCord `config.yml` 中 `priorities` 列表的第一个服务器 (例如 `lobby`)。
    * 您可以使用 BungeeCord 的 `/server` 命令切换，例如：
        * `/server lobby`
        * `/server new` (这里的 `lobby` 和 `new` 是您在 BungeeCord `config.yml` 的 `servers:` 部分定义的服务器名称)。

## 验证和故障排查

* **查看所有 Pod 状态**:
    ```bash
    kubectl get pods -n minecraft-server -w
    ```
* **查看特定 Pod 日志**:
    ```bash
    # Lobby 服务器 (mc-paper-0)
    kubectl logs -f mc-paper-0 -n minecraft-server # (假设 StatefulSet 名称为 mc-paper)
    # New 服务器 (mc-new-server-0)
    kubectl logs -f mc-new-server-0 -n minecraft-server # (假设 StatefulSet 名称为 mc-new-server)
    # BungeeCord
    kubectl logs -f $(kubectl get pods -n minecraft-server -l app=bungeecord-proxy -o jsonpath='{.items[0].metadata.name}') -n minecraft-server
    ```
* **查看 Service 状态**:
    ```bash
    kubectl get service -n minecraft-server
    ```
* **描述资源获取详细信息**:
    ```bash
    kubectl describe pod <pod-name> -n minecraft-server
    kubectl describe service <service-name> -n minecraft-server
    ```

## 伸缩

* **增加 PaperMC 服务器实例**:
    * 修改对应 StatefulSet (`01-...yaml` 或 `03-...yaml`) 中的 `spec.replicas` 字段。
    * `kubectl apply -f <statefulset_yaml_file> -n minecraft-server`
    * **注意**: 简单增加副本数会创建基于相同镜像和配置（有独立存储）的新 Pod。您还需要在 BungeeCord `config.yml` 中为这些新 Pod 定义服务器条目。
* **增加 BungeeCord 代理实例**:
    * 修改 `06-bungeecord-deployment.yaml` 中的 `spec.replicas` 字段。
    * `kubectl apply -f 06-bungeecord-deployment.yaml -n minecraft-server`

## 清理资源

* **删除单个组件**:
    ```bash
    kubectl delete -f <filename.yaml> -n minecraft-server
    # 或者按类型和名称删除
    kubectl delete statefulset mc-paper -n minecraft-server
    kubectl delete service mc-paper-headless-svc -n minecraft-server
    # ... 以此类推
    ```
* **删除所有已部署的资源 (通过删除命名空间)**:
    ```bash
    kubectl delete namespace minecraft-server
    ```
    **警告**: 此操作会删除 `minecraft-server` 命名空间中的所有内容，包括 PVC。根据您的存储回收策略，这可能导致世界数据丢失。请谨慎操作，并确保已备份重要数据。