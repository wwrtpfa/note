# 后端服务

- maven：`spring-cloud-kubernetes`

  ```xml
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-kubernetes-fabric8-all</artifactId>
  </dependency>
  ```

  

- 配置文件：bootstrap.yaml

- 修改framework，本地配置文件加载逻辑



# kubernetes

## 安装kubernetes

见部署教程

## 安装nfs服务器

```shell
# 安装nfs工具
yum install -y nfs-utils

# 创建nfs目录
mkdir -p /nfs/data

# 暴露 /nfs/data/目录  *表示任何人 括号内的表示有哪些操作
echo "/nfs/data/ *(insecure,rw,sync,no_root_squash)" > /etc/exports

# 配置生效
exportfs -r

# 开机自启 rpcbind  --now 表示现在启动
systemctl enable rpcbind --now

# 开机自启 nfs-server
systemctl enable nfs-server --now

# 防火墙配置
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-port=2049/tcp
firewall-cmd --permanent --add-port=2049/udp
firewall-cmd --reload

```



## yaml

### ingress-controller

```shell
kubectl apply -f ingress-controller.yaml
```

### namespace

```
kubectl create ns dyyun
```

### StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs
provisioner: example.com/external-nfs
parameters:
  server: 172.16.11.185
  path: /nfs/data
```



### Secret

#### secret-docker

- 登录docker，docker login
- cat ~/.docker/config.json
- 将json内容转为base64字符串

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-docker
  namespace: dyyun
data:
  .dockerconfigjson: ewogICAgImF1dGhzIjp7CiAgICAgICAgInJlZ2lzdHJ5LmNuLXNoYW5naGFpLmFsaXl1bmNzLmNvbSI6ewogICAgICAgICAgICAiYXV0aCI6IjVybVc1WTJYNmJ5TzVMaUE2SWUwNkwrYzU2ZVI1b3FBNVkrUjViR1Y1cHlKNlptUTVZV3M1WSs0T2todVpIa3VjbVZuYVhOMGNuaz0iCiAgICAgICAgfQogICAgfQp9
type: kubernetes.io/dockerconfigjson
```

#### secret-dyyun

保存所有敏感信息

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-dyyun
  namespace: dyyun
type: Opaque
data:
  # OSS
  oss.ali.bucket: ZHl5dW4tZGV2
  oss.ali.access-key-id: TFRBSTV0OHJUeGNSZW5lSFdpWjdDTHhZ
  oss.ali.access-key-secret: ejczeFl3akE0SDJVR1hNeUV6akZYQXMwQmc0TFpv
  oss.ali.endpoint: b3NzLWNuLXNoYW5naGFpLmFsaXl1bmNzLmNvbQ==
  
  # SMS
  sms.ali.access-key-id: TFRBSTV0TU1DV2RqWlk4WnRBVjNxSGVE
  sms.ali.access-key-secret: bTlzMExUWFRXc2VlZTUxYkJHYXJ5VkE2cUx6cXNp
  sms.ali.endpoint: ZHlzbXNhcGkuYWxpeXVuY3MuY29t
  
  # mysql
  mysql.username: cm9vdA==  
  mysql.password: WlBxbXFpQ0ZtVzg1
  
  # redis
  redis.password: N0ZzWFRkdlo2eWhO
  
  # mqtt
  mqtt.url: dGNwOi8vcG9zdC1jbi0ycjQyaGt1ZTYyMy5tcXR0LmFsaXl1bmNzLmNvbToxODgzCg==
  mqtt.access-key: TFRBSTV0OHJUeGNSZW5lSFdpWjdDTHhZCg==
  mqtt.secret-key: ejczeFl3akE0SDJVR1hNeUV6akZYQXMwQmc0TFpvCg==
  mqtt.instance-id: cG9zdC1jbi0ycjQyaGt1ZTYyMw==
```



### ConfigMap

#### dyyun-config

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: dyyun-config
  namespace: dyyun
data:
  logstash.host-port: 172.16.11.185:5044
  application.yml: |
    spring:      
      servlet:
        multipart:
          max-file-size: 20MB
          max-request-size: 21MB
      jackson:
        deserialization:
          read-unknown-enum-values-as-null: true  

    seata:
      tx-service-group: my_test_tx_group
      client:
        undo:
          log-serialization: kryo
      service:
        vgroup-mapping:
          my_test_tx_group: default
        grouplist:
          default: ${dyyun.seata.host-port}

    mapper:
      safe-delete: true
      safe-update: true
    mybatis:
      mapper-locations: classpath:mapper/*.xml
      
    xxl:
      job:
        admin-addresses: http://172.16.11.185:9999
        accessToken: default_token
        executor:
          app-name: ${spring.application.name}
      
    dyyun:
      # 对象存储OSS
      oss:
        provider: ali
        url-expire-time: 1h
        ali:
          bucket: ${oss.ali.bucket}
          access-key-id: ${oss.ali.access-key-id}
          access-key-secret: ${oss.ali.access-key-secret}
          endpoint: ${oss.ali.endpoint}

      # 短信SMS
      sms:
        provider: ali
        ali:
          access-key-id: ${sms.ali.access-key-id}
          access-key-secret: ${sms.ali.access-key-secret}
          endpoint: ${sms.ali.endpoint}

      # mysql
      mysql:
        host-port: 172.16.11.185:3306
        username: ${mysql.username}
        password: ${mysql.password}

      # redis
      redis:
        host: 172.16.11.185
        password: ${redis.password}
        port: 6379

      # seata
      seata:
        host-port: 172.16.11.185:8091
        
      # logstash
      logstash:
        host-port: 172.16.11.185:5044
```



#### dyyun-product-config

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: dyyun-product-config
  namespace: dyyun
data:
  application.yml: |
    dyyun:
      # mqtt
      mqtt:
        group-id: GID_DEV
        parent-topic: yun_dev
        url: ${mqtt.url}
        access-key: ${mqtt.access-key}
        secret-key: ${mqtt.secret-key}
        instance-id: ${mqtt.instance-id}
        time-to-wait: 10s
        heart-time: 10000
```



### Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-dyyun
  namespace: dyyun
  annotations:
    # 正则匹配
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  ingressClassName: nginx
  rules:
    # 运营端
    - host: ops.dyyun.mock
      http:
        paths:
          # 前端ui
          - path: /
            pathType: Prefix
            backend:
              service: 
                name: dyyun-ops-web
                port: 
                  number: 9501
          # 后端api
          - path: /api/ops
            pathType: Prefix
            backend:
              service: 
                name: dyyun-gateway
                port: 
                  number: 9000
          
    # 客户端          
    - host: dyyun.mock
      http:
        paths:
          # 前端ui
          - path: /
            pathType: Prefix
            backend:
              service: 
                name: dyyun-customer-web
                port: 
                  number: 9502
          # 后端api
          - path: /api/customer
            pathType: Prefix
            backend:
              service: 
                name: dyyun-gateway
                port: 
                  number: 9000
    # swagger
    - host: swagger.dyyun.mock
      http:
              service: 
                name: dyyun-gateway
                port: 
                  number: 9000
    # svc接口
    - host: svc.dyyun.mock
      http:
        paths:
          # auth svc      
          - path: /svc/auth
            pathType: Prefix
            backend:
              service: 
                name: dyyun-auth
                port: 
                  number: 9001
          # ops svc       
          - path: /svc/ops
            pathType: Prefix
            backend:
              service: 
                name: dyyun-ops
                port: 
                  number: 9002
          # product svc      
          - path: /svc/product
            pathType: Prefix
            backend:
              service: 
                name: dyyun-product
                port: 
                  number: 9003
          # component svc          
          - path: /svc/component
            pathType: Prefix
            backend:
              service: 
                name: dyyun-component
                port: 
                  number: 9004
          # customer svc          
          - path: /svc/customer
            pathType: Prefix
            backend:
              service: 
                name: dyyun-customer
                port: 
                  number: 9005
```



### Role & RoleBinding

下面两个配置<font color='red'>任选一个</font>

- 配置一 [Spring Cloud Kubernetes官方推荐](https://docs.spring.io/spring-cloud-kubernetes/docs/current/reference/html/#service-account)

  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    name: dyyun-reader
    namespace: dyyun
  rules:
    - apiGroups: [""]
      resources: ["configmaps", "pods", "services", "endpoints", "secrets"]
      verbs: ["get", "list", "watch"]
  
  ---
  
  kind: RoleBinding
  apiVersion: rbac.authorization.k8s.io/v1
  metadata:
    name: dyyun-reader-binding
    namespace: dyyun
  subjects:
  - kind: ServiceAccount
    name: default
    apiGroup: ""
  roleRef:
    kind: Role
    name: dyyun-reader
    apiGroup: ""
  ```

- 配置二

  ```yaml
  # 创建在pod只读角色
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
    name: dyyun-reader
    namespace: dyyun
  rules:
    - apiGroups: [""] # "" 指定核心 API 组
      resources: ["configmaps", "pods", "services", "endpoints", "secrets"]
      verbs: ["get", "list", "watch"]
  ---
  # serviceAccount binding role
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: dyyun-reader-binding
  subjects:
    - kind: ServiceAccount
      namespace: dyyun
      name: default
      apiGroup: ""
  roleRef:
    kind: ClusterRole
    name: dyyun-reader
    apiGroup: ""
  
  ```

  



### Deployment & Service

#### 后端

<font color='red'>暂时不配置优化项(cup、内存、自动扩缩等)</font>

```yaml
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: ${app_name}
  name: ${app_name}
  namespace: ${namespace}
spec:
  replicas: ${replicas} # 部署实例数
  selector:
    matchLabels:
      app: ${app_name}
  template:
    metadata:
      labels:
        app: ${app_name}
    spec:
      volumes:
      - name: secret-dyyun
        secret:
          secretName: secret-dyyun
          
      containers:
      - image: ${image}
        name: ${app_name}
        volumeMounts:
        - name: secret-dyyun
          readOnly: true
          mountPath: /etc/secrets/secret-dyyun
        env:
        - name: spring.profiles.active
          value: ${spring_profile}
        - name: spring.cloud.kubernetes.secrets.paths
          value: /etc/secrets/secret-dyyun
        - name: dyyun.log.logstash.host-port  
          valueFrom:
            configMapKeyRef:
              name: dyyun-config
              key: logstash.host-port
          
      imagePullSecrets:
      - name: secret-docker # 指定docker pull secret
---
# Service
apiVersion: v1
kind: Service
metadata:
  labels:
    app: ${app_name}
  name: ${app_name}
  namespace: ${namespace}
spec:
  ports:
  - port: ${port}
    protocol: TCP
    targetPort: ${port}
  selector:
    app: ${app_name}
```



#### 前端

```yaml
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: ${app_name}
  name: ${app_name}
  namespace: ${namespace}
spec:
  replicas: ${replicas} # 部署实例数
  selector:
    matchLabels:
      app: ${app_name}
  template:
    metadata:
      labels:
        app: ${app_name}
    spec:  
      containers:
      - image: ${image}
        name: ${app_name}
      imagePullSecrets:
      - name: secret-docker # 指定docker pull secret
---
# Service
apiVersion: v1
kind: Service
metadata:
  labels:
    app: ${app_name}
  name: ${app_name}
  namespace: ${namespace}
spec:
  ports:
  - port: ${port}
    protocol: TCP
    targetPort: 80
  selector:
    app: ${app_name}
```





# jenkins脚本

## 后端

### 打包

```shell
# 指定一些变量

# 项目工作目录
context_path=${CONTEXT_PATH}

# 应用名
project_name=dyyun-auth

# dockerfile文件位置
dockerfile_file=${context_path}/dyyun-dockerfile

# 打包好的jar位置
job_jar_file=${WORKSPACE}/${project_name}-web/target/*.jar

# ---------------------------复制的项目只需要修改上面的变量就行------------------------------

app_name=${project_name}

# image
image=${DOCKER_PREFIX_DEV}${project_name}:${BUILD_NUMBER}

# 需要使用原生cp命令 不然默认是 cp -i 会有提示
\cp ${job_jar_file} ${context_path}/${project_name}.jar

# 打包docker image
docker build -f ${dockerfile_file} --build-arg jar_file=${project_name}.jar -t ${image} ${context_path}

# 打包后删除jar
rm -f job_jar_file
rm -f ${context_path}/${project_name}.jar

# 登录阿里云docker镜像仓库
docker login -u "湖南鼎一致远科技发展有限公司" -p "${ALI_DOCKER_PASSWORD}" registry.cn-shanghai.aliyuncs.com

# 上传镜像到阿里云
docker push ${image}

# 退出阿里云docker镜像仓库
docker logout

# 删除本地打包的镜像
# docker rmi ${image}
```



### 部署

```shell
# 指定一些变量

# 项目工作目录
context_path=${CONTEXT_PATH}

# 应用名
export app_name=dyyun-auth

# spring profile
export spring_profile=dev

# 端口
export port=9001

# 实例数
export replicas=1

# 清单文件
k8s_yaml=${context_path}/kubernetes/deployment-dyyun.yaml

# ---------------------------复制的项目只需要修改上面的变量就行------------------------------

# k8s namespace
export namespace=dyyun

# image
export image=${DOCKER_PREFIX_DEV}${app_name}:${BUILD_NUMBER}

# 部署
envsubst < ${k8s_yaml} | kubectl apply -f -
```



## 前端

### 打包

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

# 删除dist目录
rm -rf ${dist_paht}
rm -rf ${context_path}/${project_name}

# 登录阿里云docker镜像仓库
docker login -u "湖南鼎一致远科技发展有限公司" -p "${ALI_DOCKER_PASSWORD}" registry.cn-shanghai.aliyuncs.com

# 上传镜像到阿里云
docker push ${docker_image_name}:${docker_image_tag}

# 退出阿里云docker镜像仓库
docker logout

# 删除本地打包的镜像
# docker rmi ${docker_image_name}:${docker_image_tag}
```



### 部署

```shell
# 指定一些变量

# 项目工作目录
context_path=${CONTEXT_PATH}

# 应用名
export app_name=dyyun-ops-web


# 端口
export port=9501

# 实例数
export replicas=1

# k8s清单文件
k8s_yaml=${context_path}/kubernetes/deployment-dyyun-web.yaml

# ---------------------------复制的项目只需要修改上面的变量就行------------------------------

# k8s namespace
export namespace=dyyun

# image
export image=${DOCKER_PREFIX_DEV}${app_name}:${BUILD_NUMBER}

# 部署
envsubst < ${k8s_yaml} | kubectl apply -f -
```





# nginx配置

