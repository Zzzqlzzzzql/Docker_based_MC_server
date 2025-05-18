# 使用官方OpenJDK镜像作为基础镜像
FROM openjdk:21-jdk

# 设置工作目录
WORKDIR /mcserver

# 下载Minecraft服务器核心镜像（根据实际版本调整URL）
RUN curl -o server.jar https://api.papermc.io/v2/projects/paper/versions/1.21/builds/130/downloads/paper-1.21-130.jar

# 下载eula.txt并自动接受协议
RUN touch eula.txt && echo "eula=true" > eula.txt

COPY server.properties /mcserver/
COPY spigot.yml /mcserver/

# 暴露Minecraft服务器端口
EXPOSE 25565

# 设置容器启动命令
CMD ["java", "-Xmx1G", "-Xms1G", "-jar", "server.jar", "nogui"]