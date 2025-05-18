# 创建镜像
docker build --no-cache -t 1_21 .
# 运行镜像
docker run -it -p 25535:25535 1_21:latest