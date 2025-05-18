# 使用 Alpine Linux 作为基础镜像
FROM alpine:latest

# 设置工作目录
WORKDIR /bungeecord

# 安装必要的依赖
RUN apk add --no-cache openjdk17-jre

# 下载 BungeeCord
RUN wget -O BungeeCord.jar https://ci.md-5.net/job/BungeeCord/lastSuccessfulBuild/artifact/bootstrap/target/BungeeCord.jar

# 创建配置文件或等待生成配置
COPY config.yml /bungeecord/

# 暴露端口
EXPOSE 25577

# 设置启动命令
CMD ["java", "-Xms128M", "-Xmx512M", "-jar", "BungeeCord.jar"]