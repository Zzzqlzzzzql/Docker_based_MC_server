# 地址设置
查看服务器地址，并修改config.yml文件
# 防火墙设置
允许服务器端口和代理监听端口转发  
firewall-cmd --permanent --add-port=25535/tcp  
firewall-cmd --permanent --add-port=25565/tcp  
firewall-cmd --permanent --add-port=25577/tcp  
firewall-cmd --reload
# 创建镜像
docker build --no-cache -t bungeecord .
# 运行镜像
docker run -it -p 25577:25577 bungeecord:latest