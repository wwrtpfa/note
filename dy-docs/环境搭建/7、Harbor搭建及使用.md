# 简介

​		它以 Registry 为基础，提供了对用户友好的管理界面，可以帮助我们快速搭建一个企业级的 Docker Registry 服务。Harbor 的每个组件都是以 Docker 容器的形式构建的，使用 Docker Compose 进行部署。



# 前置条件

docker 、docker-compose

安装参考：[1、基础搭建.md](1、基础搭建.md)



# 安装Harbor

## 在线安装

联网的情况下使用在线安装

官方文档：[https://goharbor.io/docs/2.7.0/install-config/](https://goharbor.io/docs/2.7.0/install-config/)

下载地址：[https://github.com/goharbor/harbor/releases](https://github.com/goharbor/harbor/releases)

如果开启自启失败，请参考[https://cloud.tencent.com/developer/article/2159179](https://cloud.tencent.com/developer/article/2159179)

下载 `harbor-online-installer-<version>.tgz`



```shell
# 解压
tar -zxvf harbor-online-installer-v2.5.6.tgz

# 复制配置文件模板
cp harbor/harbor.yml.tmpl harbor/harbor.yml

# 修改配置文件,仔细查看！！！
# 1、hostname
# 2*、看情况配置http或https，如果不配置https，则将其注释
# 3、如果需要配置外部redis，添加external_redis.host和external_redis.password进行配置，其它配置参考官方文档
# 4、默认密码可在GUI界面修改
# 5*、修改挂载路径 data_volume: /home/harbor_data
# 注意！！！
# 1、如果已有名称为nginx的容器，则需要先将其rename或remove，否则会安装失败。
# 2、如果没有配置外部redis，则会创建名为redis的容器，如果已有名称为redis的容器，则需要先将其rename或remove，否则会安装失败。

vim harbor/harbor.yml

# 执行安装脚本
sh harbor/install.sh

# 安装完毕后可根据配置文件的内容自行输入url访问
# 账号：admin
# 密码：见配置文件harbor_admin_password，默认为 Harbor12345
# 安装完毕之后可重命名nginx容器
docker rename nginx harbor-nginx

# 如果没有配置https,docker login登录不上
# 解决方案1
# 可以在/etc/docker/daemon.json加入"insecure-registries":["ip:port"]，然后
# sudo systemctl daemon-reload
# sudo systemctl restart docker

# 解决方案2
# 配置https
```

