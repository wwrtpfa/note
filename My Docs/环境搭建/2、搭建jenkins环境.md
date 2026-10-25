# 基于docker安装jenkins

```shell
# 找一个比较大的盘挂载jenkins_home，因为jenkins一般需要10G以上的磁盘空间
df -h

# 创建jenkins_home挂载目录
# mkdir /home/jenkins_home

# 赋权，否则挂载之后无权限会报错
# chmod -R 777 /home/jenkins_home

# 创建云平台工作目录
# mkdir /home/dyyun
# chmod -R 777 /home/dyyun

# https://hub.docker.com/r/jenkins/jenkins，找到最新的版本，docker pull latest并不一定会拉最新的

# 创建maven仓库
# mkdir -p /home/maven/repository

# 拉取镜像
docker pull jenkins/jenkins:2.355

# 运行，后面两个docker的挂载是为了在 jenkins容器中使用宿主机的docker,-u root拥有root权限，就不需要修改挂载目录的权限了
docker run -d --restart=always --name jenkins \
-p 8899:8080 -p 50000:50000 \
-u root \
--env JAVA_OPTS=-Xmx6g \
-v /home/jenkins_context:/var/jenkins_context \
-v /home/jenkins_home:/var/jenkins_home \
-v /home/maven/repository:/home/maven/repository \
-v $(which docker):/usr/bin/docker \
-v /var/run/docker.sock:/var/run/docker.sock \
jenkins/jenkins:2.355
```



# 开始使用

## 1、输入初始密码，点击继续

获取初始密码：宿主机执行 `cat /home/jenkins_home/secrets/initialAdminPassword`

或者 `docker logs -f jenkins` 在日志中有显示

## 2、安装插件

### Folders（必需）

该插件允许用户创建“文件夹”来组织作业。用户可以定义自定义分类法（例如，按项目类型，组织类型）。文件夹是可嵌套的，您可以在文件夹中定义视图。

### OWASP Markup Formatter(必需)

Jenkins**允许具有适当权限的用户输入各种对象的描述，例如视图、作业、构建等**。这些描述由标记格式化程序过滤。它们有两个目的： 允许用户对这些描述使用丰富的格式。

### Build Timeout(必需)

这个插件允许构建在指定的时间过去后自动终止。

### Timestamper(必需)
将时间戳添加到控制台输出

### Workspace Cleanup(必需)

该插件提供了一个构建包装器（**在构建开始前删除工作区**）和一个构建后步骤（**在构建完成时删除工作区**）。这些步骤允许您配置将在什么情况下删除哪些文件。构建后步骤也可以考虑构建状态。

### Ant

向Jenkins添加Apache Ant支持

### Gradle

该插件允许Jenkins直接调用Gradle构建脚本。

### Pipeline(必需)

Pipeline 等价于流水线任务

Jenkins Pipeline（或简称为“Pipeline”）是**一套插件，支持将持续交付管道实现和集成到 Jenkins**中。持续交付管道是您将软件从版本控制到用户和客户的过程的自动化表达。

### Pipeline: Stage View(必需)
可以查看任务每个阶段的执行视图

### Git(必需)

该插件将Git与Jenkins集成在一起。

### SSH Build Agents

允许使用SSH协议的Java实现在SSH上启动代理。

### Matrix Authorization Strategy(必需)
提供基于矩阵的安全授权策略

### PAM Authentication(必需)
向Jenkins添加Unix插座身份验证模块（PAM）支持

允许启用、禁用和配置 Jenkins 的关键功能。PAM 身份验证：添加可插入身份验证模块 (PAM) 支持。“这是**一种将多个低级身份验证方案集成到高级应用程序编程接口（API）中的机制**”

### LDAP

将LDAP身份验证添加到Jenkins

### Email Extension

此插件允许您配置电子邮件通知的各个方面。您可以自定义电子邮件的发送时间、接收人以及电子邮件的内容。

### Mailer

此插件允许您为构建结果配置电子邮件通知。

### Localization: Chinese (Simplified) (必需)

Jenkins 核心和插件的简体中文本地化。

### Locale

这个插件控制 Jenkins 的语言。

通常，如果首选语言的翻译可用，Jenkins 会尊重浏览器的语言首选项，并在构建期间使用系统默认语言环境来获取消息。该插件允许您：

- 将系统默认语言环境覆盖为您选择的语言
- 完全忽略浏览器的语言偏好

此功能有时对多语言环境很方便。

### Credentials Binding(推荐)

允许将凭据绑定到环境变量，以便在其他构建步骤中使用。

### Maven Integration(必需)

> 需要在安装好之后才能搜索进行安装，推荐插件没有这个

maven集成

### SSH(必需)

SSH远程执行脚本

### NodeJS Plugin(必需)

Nodejs插件将NodeJS脚本作为构建步骤

### Role-based Authorization Strategy(必需)

使用基于角色的策略启用用户授权。可以在全局定义角色，也可以针对正则表达式选择的特定作业或节点定义



### Config File Provider(必需)

能够提供配置文件(例如maven的settings.xml, XML, groovy，自定义文件，…)通过UI加载，这些文件将被复制到作业工作区。 





## 3、创建第一个管理员用户

填好表单，完成之后进入Jenkins

用户名：admin

密码：F1OpDEUz6Ztq

全名：admin

邮箱：admin@hndingyi.com.cn



进入jenkins后安装插件`Maven Integration`


