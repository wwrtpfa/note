# 搭建xxl-job-admin

[github地址](https://github.com/xuxueli/xxl-job)



## 数据库配置

创建数据库

库名：xxl_job

编码：utf8mb4

初始化脚本：[tables_xxl_job.sql](https://github.com/xuxueli/xxl-job/blob/master/doc/db/tables_xxl_job.sql)



## 通过源码部署

- 源码：[https://github.com/xuxueli/xxl-job](https://github.com/xuxueli/xxl-job)

- 配置：`xxl-job-admin/application.properties`

- maven进行打包




## docker 部署

如需指定版本，先查找tag [https://hub.docker.com/r/xuxueli/xxl-job-admin/](https://hub.docker.com/r/xuxueli/xxl-job-admin/)

```shell
docker pull xuxueli/xxl-job-admin:2.3.1

# 日志挂载 -v /var/log/xxl-job:/data/applogs
docker run -d --restart=always --name xxl-job-admin \
-p 9999:8080 \
-e JAVA_OPTS="-Xmx512m -Xms256m" \
-e PARAMS="\
--server.servlet.context-path=/ \
--spring.datasource.url=jdbc:mysql://172.16.11.185:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai \
--spring.datasource.username=root \
--spring.datasource.password=ZPqmqiCFmW85" \
xuxueli/xxl-job-admin:2.3.1
```

浏览器打开：`http://ip:9999` 进行访问

admin/123456



# 配置任务

## spring job handler配置

