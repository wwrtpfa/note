# 简介

轻量级和可移植的命令行 YAML 处理器

注意：本项目下的所有yaml都通过yq进行操作，需要在集群的control-plane节点进行安装

# 安装

可以参考[github](https://github.com/mikefarah/yq/#install)

下载安装包 [https://github.com/mikefarah/yq/releases](https://github.com/mikefarah/yq/releases)

如: [yq_linux_amd64](https://github.com/mikefarah/yq/releases/download/v4.27.5/yq_linux_amd64)

```shell
# 进行科学上网下载，再上传到服务器
mv yq_linux_amd64 /usr/bin/yq
chmod +x /usr/bin/yq

# 检查是否可用
yq -V
```



# 使用方法

[官方文档](https://mikefarah.gitbook.io/yq/operators)



## 示例

- test1.yaml

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx1
  spec:
    selector:
      matchLabels:
        app: ${app}
    template:
      metadata:
        labels:
          app: ${app}
      spec:
        containers:
        - image: ${image}
          name: nginx
  ```

- test2.yaml

  ```yaml
  metadata:
    name: nginx2
    labels:
      app: ${app}
    namespace: ${namespace}
  spec:
    replicas: 2
  ```

将test2.yaml合并到test1.yaml，并替换变量

```yaml
image="nginx:1" app=nginx namespace=test yq '. *= load("test2.yaml") | (.. | select(tag == "!!str")) |= envsubst' test1.yaml
```

