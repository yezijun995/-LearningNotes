# docker下nacos的单击配置并连接数据库

> 有时我们想要在 docker 下配置单机的nacos，但是又不想用 nacos 默认的数据库，这是就需要切换到mysql（目前 nacos 只支持mysql）
>



```bash
# 先下载相应的 nacos 镜像
docker pull nacos-server

# 下载完毕之后，配置清楚mysql，以及账户与密码、端口号等，然后进行一下配置，具体修改为自己IP地址，端口号，账户密码等。
docker run --env MODE=standalone --name mynacos -d -p 8848:8848 -e MYSQL_SERVICE_HOST=192.168.106.131  -e MYSQL_SERVICE_PORT=3306  -e MYSQL_SERVICE_DB_NAME=nacos_config  -e MYSQL_SERVICE_USER=root  -e MYSQL_SERVICE_PASSWORD=123456  -e SPRING_DATASOURCE_PLATFORM=mysql  -e MYSQL_DATABASE_NUM=1 nacos/nacos-server
```

