# 全局配置



## 全局安全配置

授权策略：Role-Based Strategy (需要安装插件)



## Manage and Assign Roles

### Manage Roles

<img src="http://cdn.pengfanao.top/16djD.png" alt="16djD.png" border="0">

### Assign Roles

<img src="http://cdn.pengfanao.top/16VWF.png" alt="16VWF.png" border="0">



## 全局工具配置

### Global Maven settings.xml

Manage Jenkins --> Managed files -> Add a new Config

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings
    xmlns="http://maven.apache.org/SETTINGS/1.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <localRepository>/home/maven/repository</localRepository>
    <mirrors>
        <mirror>
            <id>alimaven</id>
            <name>aliyun maven</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
            <mirrorOf>central</mirrorOf>
        </mirror>
    </mirrors>
    <profiles></profiles>
</settings>
```



### Maven settings.xml

Manage Jenkins --> Managed files -> Add a new Config

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings
    xmlns="http://maven.apache.org/SETTINGS/1.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <localRepository>/home/maven/repository</localRepository>
    <servers>
        <server>
            <id>hndyzy-yun-dyyun-maven</id>
            <username>maven-1655188197585</username>
            <password>346d9f3d8ef43558a5b0a25d9d97d4d53d0de753</password>
        </server>
    </servers>
    <mirrors>
        <mirror>
            <id>nexus-tencentyun</id>
            <mirrorOf>!hndyzy-yun-dyyun-maven</mirrorOf>
            <name>Nexus tencentyun</name>
            <url>http://mirrors.cloud.tencent.com/nexus/repository/maven-public/</url>
        </mirror>
    </mirrors>
    <profiles>
        <profile>
            <id>Repository Proxy</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <repositories>
                <repository>
                    <id>hndyzy-yun-dyyun-maven</id>
                    <name>maven</name>
                    <url>https://hndyzy-yun-maven.pkg.coding.net/repository/dyyun/maven/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </repository>
            </repositories>
        </profile>
    </profiles>
</settings>
```



### JDK

docker安装的jenkins自带JDK，无需安装



### Git

docker安装的jenkins自带git，无需安装



### maven

maven:  apache-maven-3.6.3

自动安装，选择 3.6.3

### node js

node js: node js 16.14.2

自动安装，选择 16.14.2



## 系统配置

### 全局属性-环境变量

```properties
ALI_DOCKER_PASSWORD=Hndy.registry
CONTEXT_PATH=/home/dyyun
DOCKER_PREFIX_DEV=registry.cn-shanghai.aliyuncs.com/yun-dev/
```

### SSH remote hosts

需要安装插件 ssh





# 任务配置

视图：添加dev



## 公共配置（后端）

任务类型：**maven项目**

### git

- url:  `https://e.coding.net/hndyzy-yun/dyyun/dyyun-gateway.git`

- branches:  `*/develop`

###  构建触发器

去掉所有构建触发器的勾



### 构建环境

勾上 Add timestamps to the Console Output

勾上 Terminate a build if it's stuck  设置10分钟超时（首次构建不勾选）

### Pre Steps

- shell

  ```shell
  # 删除maven仓库中*.lastUpdated的文件
  find /home/maven/repository -name "*.lastUpdated" -exec grep -q "Could not transfer" {} \; -print -exec rm {} \;
  ```

### Build

- maven命令:  `clean install -U -DskipTests`

- 高级：

  去掉 Enable triggering of downstream projects 的勾

  选择maven setting



## dev-dyyun-gateway（后端-无模块）

### Post Steps （Run only if build succeeds）

- shell

  ```shell
  # 指定一些变量
  
  # 项目工作目录
  context_path=${CONTEXT_PATH}
  
  # 项目名
  project_name=dyyun-gateway
  
  # 设置spring profile
  spring_profile=dev
  
  # yaml 配置文件
  yaml=dyyun-${spring_profile}.yaml
  
  # 打包好的jar位置
  job_jar_file=${WORKSPACE}/target/*.jar
  
  # ---------------------------复制的项目只需要修改上面的变量就行------------------------------
  
  # dockerfile文件位置
  dockerfile_file=${context_path}/dyyun-dockerfile
  
  # docker镜像名
  docker_image_name=${DOCKER_PREFIX_DEV}${project_name}
  
  # docker镜像tag
  docker_image_tag=${BUILD_NUMBER}
  
  # 需要使用原生cp命令 不然默认是 cp -i 会有提示
  \cp ${job_jar_file} ${context_path}/${project_name}.jar
  
  
  # 打包docker image
  docker build -f ${dockerfile_file}  --build-arg yaml_file=dyyun-${spring_profile}.yaml --build-arg jar_file=${project_name}.jar -t ${docker_image_name}:${docker_image_tag} ${context_path}
  
  # 打包后删除jar
  rm -f job_jar_file
  rm -f ${context_path}/${project_name}.jar
  
  # 登录阿里云docker镜像仓库
  docker login -u "湖南鼎一致远科技发展有限公司" -p "${ALI_DOCKER_PASSWORD}" registry.cn-shanghai.aliyuncs.com
  
  # 上传镜像到阿里云
  docker push ${docker_image_name}:${docker_image_tag}
  
  # 退出阿里云docker镜像仓库
  docker logout
  
  # 删除本地打包的镜像
  docker rmi ${docker_image_name}:${docker_image_tag}
  ```

- ssh

  ```shell
  # 指定一些变量
  
  # 项目工作目录
  context_path=${CONTEXT_PATH}
  
  # 设置端口
  port=9000
  
  # 日志路径
  logs_path=${context_path}/logs
  
  # 项目名
  project_name=dyyun-gateway
  
  # 设置spring profile
  spring_profile=dev
  
  # 本地docker镜像保留数量
  docker_image_retain_number=3
  
  # ---------------------------复制的项目只需要修改上面的变量就行------------------------------
  
  # docker镜像名
  docker_image_name=${DOCKER_PREFIX_DEV}${project_name}
  
  # docker容器名
  docker_container_name=${project_name}
  
  # docker镜像tag
  docker_image_tag=${BUILD_NUMBER}
  
  # 登录阿里云docker镜像仓库
  docker login -u "湖南鼎一致远科技发展有限公司" -p "${ALI_DOCKER_PASSWORD}" registry.cn-shanghai.aliyuncs.com
  
  # 拉取镜像
  docker pull ${docker_image_name}:${docker_image_tag}
  
  # 镜像拉取后退出登录
  docker logout
  
  # 获取容器ID
  container_id=`docker inspect --format="{{.Id}}" ${docker_container_name}`
  
  # 判断容器是否存在
  if [ "$container_id"x != ""x ]; then
  	# 获取是否在运行
  	is_runing=`docker inspect --format="{{.State.Running}}" ${container_id}`
  	
  	if [ "$is_runing"x != ""x ]; then
  		# 容器正在运行，停止	
  		docker stop $container_id
  	fi
  	# 最后删除容器
  	docker rm $container_id
  fi
  
  # run docker镜像
  docker run -d --name ${docker_container_name} --restart always -p 127.0.0.1:$port:$port -v ${logs_path}:${logs_path} ${docker_image_name}:${docker_image_tag} --spring.profiles.active=$spring_profile
  
  
  
  # 最后进行镜像的清理
  
  # 删除过期镜像，保留最近的num个版本
  retain_number=$docker_image_retain_number
  
  # 当前时间
  current_time=`date '+%Y-%m-%d_%H:%M:%S'`
  
  # 需要清理的镜像名，全名
  clear_image_name=$docker_image_name
  
  # 计数
  cnt=1
  
  # 清理日志的文件位置
  remove_docker_image_log_file=${logs_path}/remove-docker-images.log
  
  # 查询按名字查询image列表，去掉第一行(也就是表头) 并且取第二列 TAG 再进行倒序排序 
  for tag in `docker images $clear_image_name| awk 'NR != 1 {print $2}' |sort -nr`
  do
  	if [ $cnt -gt $retain_number ] ; then
  		echo "$current_time remove docker image $clear_image_name:$tag" >> $remove_docker_image_log_file
  		docker rmi $clear_image_name:$tag
  	fi
  	cnt=`expr $cnt + 1`
  done
  ```



## dev-dyyun-auth（后端-标准模块）

### Post Steps

- shell

  ```shell
  
  # 指定一些变量
  
  # 项目工作目录
  context_path=${CONTEXT_PATH}
  
  # 项目名
  project_name=dyyun-auth
  
  # 设置spring profile
  spring_profile=dev
  
  
  # 打包好的jar位置
  job_jar_file=${WORKSPACE}/${project_name}-web/target/*.jar
  
  # ---------------------------复制的项目只需要修改上面的变量就行------------------------------
  
  
  # dockerfile文件位置
  dockerfile_file=${context_path}/dyyun-dockerfile
  
  # docker镜像名
  docker_image_name=${DOCKER_PREFIX_DEV}${project_name}
  
  # docker镜像tag
  docker_image_tag=${BUILD_NUMBER}
  
  # 需要使用原生cp命令 不然默认是 cp -i 会有提示
  \cp ${job_jar_file} ${context_path}/${project_name}.jar
  
  
  # 打包docker image
  docker build -f ${dockerfile_file}  --build-arg yaml_file=dyyun-${spring_profile}.yaml --build-arg jar_file=${project_name}.jar -t ${docker_image_name}:${docker_image_tag} ${context_path}
  
  # 打包后删除jar
  rm -f job_jar_file
  rm -f ${context_path}/${project_name}.jar
  
  
  # 登录阿里云docker镜像仓库
  docker login -u "湖南鼎一致远科技发展有限公司" -p "${ALI_DOCKER_PASSWORD}" registry.cn-shanghai.aliyuncs.com
  
  # 上传镜像到阿里云
  docker push ${docker_image_name}:${docker_image_tag}
  
  # 退出阿里云docker镜像仓库
  docker logout
  
  # 删除本地打包的镜像
  docker rmi ${docker_image_name}:${docker_image_tag}
  
  ```

- ssh

  ```shell
  # 指定一些变量
  
  # 项目工作目录
  context_path=${CONTEXT_PATH}
  
  # 设置端口
  port=9001
  
  # 日志路径
  logs_path=${context_path}/logs
  
  # 项目名
  project_name=dyyun-auth
  
  # 设置spring profile
  spring_profile=dev
  
  # 本地docker镜像保留数量
  docker_image_retain_number=3
  
  # ---------------------------复制的项目只需要修改上面的变量就行------------------------------
  
  # docker镜像名
  docker_image_name=${DOCKER_PREFIX_DEV}${project_name}
  
  # docker容器名
  docker_container_name=${project_name}
  
  # docker镜像tag
  docker_image_tag=${BUILD_NUMBER}
  
  # 登录阿里云docker镜像仓库
  docker login -u "湖南鼎一致远科技发展有限公司" -p "${ALI_DOCKER_PASSWORD}" registry.cn-shanghai.aliyuncs.com
  
  # 拉取镜像
  docker pull ${docker_image_name}:${docker_image_tag}
  
  # 镜像拉取后退出登录
  docker logout
  
  # 获取容器ID
  container_id=`docker inspect --format="{{.Id}}" ${docker_container_name}`
  
  # 判断容器是否存在
  if [ "$container_id"x != ""x ]; then
  	# 获取是否在运行
  	is_runing=`docker inspect --format="{{.State.Running}}" ${container_id}`
  	
  	if [ "$is_runing"x != ""x ]; then
  		# 容器正在运行，停止	
  		docker stop $container_id
  	fi
  	# 最后删除容器
  	docker rm $container_id
  fi
  
  # run docker镜像
  docker run -d --name ${docker_container_name} --restart always -p 127.0.0.1:$port:$port -v ${logs_path}:${logs_path} ${docker_image_name}:${docker_image_tag} --spring.profiles.active=$spring_profile
  
  
  
  # 最后进行镜像的清理
  
  # 删除过期镜像，保留最近的num个版本
  retain_number=$docker_image_retain_number
  
  # 当前时间
  current_time=`date '+%Y-%m-%d_%H:%M:%S'`
  
  # 需要清理的镜像名，全名
  clear_image_name=$docker_image_name
  
  # 计数
  cnt=1
  
  # 清理日志的文件位置
  remove_docker_image_log_file=${logs_path}/remove-docker-images.log
  
  # 查询按名字查询image列表，去掉第一行(也就是表头) 并且取第二列 TAG 再进行倒序排序 
  for tag in `docker images $clear_image_name| awk 'NR != 1 {print $2}' |sort -nr`
  do
  	if [ $cnt -gt $retain_number ] ; then
  		echo "$current_time remove docker image $clear_image_name:$tag" >> $remove_docker_image_log_file
  		docker rmi $clear_image_name:$tag
  	fi
  	cnt=`expr $cnt + 1`
  done
  
  ```



## dyyun-ops-web（前端）

任务类型：**自由风格的软件项目**

### git

- url:  `https://e.coding.net/hndyzy-yun/dyyun/dyyun-ops-web.git`

- branches:  `*/develop`

### 构建环境

- 勾选

  Add timestamps to the Console Output
  
  Provide Node & npm bin/ folder to PATH

### Build

- shell

  ```shell
  # npm build 脚本
  node -v
  npm -v
  npm install
  npm run build
  ```

- shell

  ```shell
  # 打包为docker镜像
  
  # 项目工作目录
  context_path=${CONTEXT_PATH}
  
  # 项目名
  project_name=dyyun-ops-web
  
  # 打包好的dist路径
  dist_paht=${WORKSPACE}/dist
  
  # dockerfile文件位置
  dockerfile_file=${context_path}/dyyun-dockerfile-web
  
  # ---------------------------复制的项目只需要修改上面的变量就行------------------------------
  
  # docker镜像名
  docker_image_name=${DOCKER_PREFIX_DEV}${project_name}
  
  # docker镜像tag
  docker_image_tag=${BUILD_NUMBER}
  
  # 移动dist目录
  \cp -r ${dist_paht} ${context_path}/${project_name}
  
  
  # 打包docker image
  docker build -f ${dockerfile_file} --build-arg DIST_PATH=${project_name} -t ${docker_image_name}:${docker_image_tag} ${context_path}
  
  # 登录阿里云docker镜像仓库
  docker login -u "湖南鼎一致远科技发展有限公司" -p "${ALI_DOCKER_PASSWORD}" registry.cn-shanghai.aliyuncs.com
  
  # 上传镜像到阿里云
  docker push ${docker_image_name}:${docker_image_tag}
  
  # 退出阿里云docker镜像仓库
  docker logout
  
  # 删除本地打包的镜像
  docker rmi ${docker_image_name}:${docker_image_tag}
  
  # 删除dist目录
  rm -rf ${dist_paht}
  rm -rf ${context_path}/${project_name}
  ```

- ssh

  ```shell
  # 指定一些变量
  
  # 日志路径
  logs_path=${CONTEXT_PATH}/logs
  
  # 设置端口
  port=9501
  
  # 项目名
  project_name=dyyun-ops-web
  
  # 本地docker镜像保留数量
  docker_image_retain_number=3
  
  # ---------------------------复制的项目只需要修改上面的变量就行------------------------------
  
  # docker镜像名
  docker_image_name=${DOCKER_PREFIX_DEV}${project_name}
  
  # docker容器名
  docker_container_name=${project_name}
  
  # docker镜像tag
  docker_image_tag=${BUILD_NUMBER}
  
  # 登录阿里云docker镜像仓库
  docker login -u "湖南鼎一致远科技发展有限公司" -p "${ALI_DOCKER_PASSWORD}" registry.cn-shanghai.aliyuncs.com
  
  # 拉取镜像
  docker pull ${docker_image_name}:${docker_image_tag}
  
  # 镜像拉取后退出登录
  docker logout
  
  # 获取容器ID
  container_id=`docker inspect --format="{{.Id}}" ${docker_container_name}`
  
  # 判断容器是否存在
  if [ "$container_id"x != ""x ]; then
  	# 获取是否在运行
  	is_runing=`docker inspect --format="{{.State.Running}}" ${container_id}`
  	
  	if [ "$is_runing"x != ""x ]; then
  		# 容器正在运行，停止	
  		docker stop $container_id
  	fi
  	# 最后删除容器
  	docker rm $container_id
  fi
  
  # run docker镜像
  docker run -d --name ${docker_container_name} --restart always -p $port:80  ${docker_image_name}:${docker_image_tag}
  
  # 最后进行镜像的清理
  
  # 删除过期镜像，保留最近的num个版本
  retain_number=$docker_image_retain_number
  
  # 当前时间
  current_time=`date '+%Y-%m-%d_%H:%M:%S'`
  
  # 需要清理的镜像名，全名
  clear_image_name=$docker_image_name
  
  # 计数
  cnt=1
  
  # 清理日志的文件位置
  remove_docker_image_log_file=${logs_path}/remove-docker-images.log
  
  # 查询按名字查询image列表，去掉第一行(也就是表头) 并且取第二列 TAG 再进行倒序排序 
  for tag in `docker images $clear_image_name| awk 'NR != 1 {print $2}' |sort -nr`
  do
  	if [ $cnt -gt $retain_number ] ; then
  		echo "$current_time remove docker image $clear_image_name:$tag" >> $remove_docker_image_log_file
  		docker rmi $clear_image_name:$tag
  	fi
  	cnt=`expr $cnt + 1`
  done
  
  ```

  



