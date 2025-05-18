# 创建镜像
docker build --no-cache -t 1_21base .
# 运行镜像
docker run -it -p 25565:25565 1_21base:latest