# fogcloud部署

#### foglcoud相关服务镜像

```
ghcr.io/fogcloud-io/fogcloud:latest
ghcr.io/fogcloud-io/fogcloud-web:latest
ghcr.io/fogcloud-io/faas-builder:latest
ghcr.io/fogcloud-io/fogcron-scheduler:latest
```

#### 配置要求
* 一台或多台linux-amd64主机，建议三台或以上；
* 单机最低配置：2核cpu，4g内存，40g硬盘；建议：4核cpu，8g内存，40g硬盘；
* 主机间网络互通；
* 主机可以访问外网；
* 主机部分端口可以外网或局域网内访问；

fogcloud默认使用的端口：

| 端口 | 描述 | 备注 |
| --- | --- | --- |
| 80 | http服务 | 因为部分设备因为性能问题，无法访问https服务，因此使用http服务用于设备固件下载 |
| 443 | https服务 | 管理后台 |
| 1883 | mqtt-tcp服务 | 用于设备不安全mqtt连接 |
| 8883 | mqtt-ssl-tcp服务 | 用于设备mqtt连接 |
| 8083 | mqtt-websocket服务 | 用于前端虚拟设备不安全连接 |
| 8084 | mqtt-websocket-ssl服务 | 用于前端虚拟设备安全连接 |
| 5672 | amqp服务 | 用于服务端消息订阅（未加密） |
| 5671 | amqp-tls服务 | 用于服务端消息订阅 |

* 推荐使用helm部署fogcloud

## 1 使用docker-compose部署

### 1.1 安装前准备

- [安装 docker](https://docs.docker.com/engine/install/) 20+

### 1.2 安装
```bash
git clone https://github.com/fogcloud-io/fogcloud-deploy.git
cd docker
docker compose up -d
```

### 1.3 卸载
```bash
docker compose down
```

## 2 使用kubernetes部署

### 2.1 安装前准备
- [安装 kubernetes](https://docs.k3s.io/installation) 1.19+ 
- [安装并设置 kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) 1.19+

### 2.2 安装
```bash
git clone https://github.com/fogcloud-io/fogcloud-deploy.git
cd kubernetes
kubectl create ns fogcloud
./create-configmap.sh
kubectl apply -f fogcloud-all.yaml
```

### 2.3 卸载
```bash
kubectl delete -f fogcloud-all.yaml
```

## 3 使用helm部署

### 3.1 安装前准备

- [安装 kubernetes](https://docs.k3s.io/installation) 1.19+ 
- [安装并设置 kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) 1.19+
- [安装 helm](https://helm.sh/docs/intro/install/) 3+
- [使用helm安装 fission](https://fission.io/docs/installation/#with-helm)

### 3.2 获取chart

```console
helm repo add fogcloud-charts https://fogcloud-io.github.io/fogcloud-charts
helm repo update
helm pull fogcloud-charts/fogcloud-charts --untar
```
运行后在当前目录会生成fogcloud-charts文件夹

### 3.3 安装chart
1. 拷贝fogcloud-charts目录的values.yaml文件，并命名为myvalues.yaml
2. 编辑myvalues.yaml文件，参考配置说明
3. 安装fogcloud-charts
```console
export NAMESPACE_NAME=fogcloud
export RELEASE_NAME=fogcloud
kubectl create namespace ${NAMESPACE_NAME}
helm install -f myvalues.yaml ${RELEASE_NAME} --set namespace=${NAMESPACE_NAME} ./fogcloud-charts
```
4. 升级fogcloud-charts
```console
helm upgrade -f myvalues.yaml ${RELEASE_NAME} ./fogcloud-charts
```

### 3.4 卸载chart

```console
helm uninstall ${RELEASE_NAME}
```
注意：默认启用了helm的资源保留，卸载时不会释放persistent volume资源；

### 3.5 配置说明：

| 配置项 | 说明 | 默认值 |
| --- | --- | --- |
| `namespace` | 应用所在命名空间 | `fogcloud` |
| `environment` | 配置文件的环境 | `production` |
| `imagePullPolicy` | 镜像拉取策略 | `Always` | 
| `k8sApiServer` | k8s server api地址，用来创建k8s StatefulSet资源；可以通过`kubectl config view`获取 | `https://localhost:6443` |
| `fissionEnabled` | 是否启用了云函数功能 | `false` |
| **expose** | | | 
| `expose.type` | 如何暴露服务：`Ingress`、`ClusterIP`、`NodePort`或`LoadBalancer`，其他值将被忽略，服务的创建将被跳过。| `ClusterIP` |
| `expose.hosts.webAdmin` | web服务域名，使用ingress暴露服务时会用到 | `localhost` |
| `expose.hosts.api` | api服务域名，使用ingress暴露服务时会用到 | `localhost` | 
| `expose.hosts.mqtt` | mqtt服务域名 | `localhost` | 
| `expose.hosts.amqp` | amqp服务域名 | `localhost` | 
| `expose.tls.enabled` | 是否启用ingress tls | `false` |
| `expose.tls.cert.api.certSource` | api服务证书的来源：`file`, `auto`或`none`；1）`file`：使用证书文件，直接将*.key和*.crt文件放入fogcloud-charts/config/cert/api目录下；2）`auto`：使用cert-manager生成免费证书，需要保证设置的域名可用；3）`none`：不为服务入口配置证书 | `none` |
| `expose.tls.cert.api.secretName` | api服务所用证书对应的k8s secret资源名 | `fogcloud-api` |
| `expose.tls.cert.webAdmin.certSource` | web服务证书的来源：`file`, `auto`或`none`；1）`file`：使用证书文件，直接将*.key和*.crt文件放入fogcloud-charts/config/cert/api目录下；2）`auto`：使用cert-manager生成免费证书，需要保证设置的域名可用；3）`none`：不为服务入口配置证书 | `none` |
| `expose.tls.cert.webAdmin.secretName` | web服务所用证书对应的k8s secret资源名 | `fogcloud-web` |
| `expose.Ingress` | `expose.type`设置为Ingress时才需要设置 | |
| `expose.Ingress.className` | ingress class资源名| `traefik` |
| `expose.Ingress.controller` | ingress controller类型 | `traefik.io/ingress-controller` |
| `expose.Ingress.annotations` | ingress注释，可以用来设置ingress部分参数 | `{}` |
| `expose.NodePort` |  |  |
| `expose.NodePort.externalTrafficPolicy` | 流量策略：`Cluster`或`Local`；1）`Cluster`：流量可以转发到其他k8s节点的pod，2）`Local`：流量只转发给本机的pod| `Local` | 
| `expose.NodePort.ports.webAdmin.httpPort` | web服务的NodePort端口，可用于外网暴露web服务 | `8888` |
| `expose.NodePort.ports.api.httpPort` | api服务的NodePort端口，可用于外网暴露api服务 | `8000` |
| `expose.LoadBalancer` | | |
| `expose.LoadBalancer.externalTrafficPolicy` | 流量策略：`Cluster`或`Local`；1）`Cluster`：流量可以转发到其他k8s节点的pod，2）`Local`：流量只转发给本机的pod | `Local` |
| `expose.LoadBalancer.ports.webAdmin.healthCheckNodePort` | 健康检查端口，用于外部slb检测web服务是否正常运行 | `8880` |
| `expose.LoadBalancer.ports.api.healthCheckNodePort` | 健康检查端口，用于外部slb检测api服务是否正常运行 | `8880` |
| **secret** | | |
| `secret.imageCredentials` | 配置私有镜像仓库源 | |
| `secret.imageCredentials[].registry` | 私有镜像仓库地址 | `""` | 
| `secret.imageCredentials[].name` | k8s dockerconfigjson secret名称 | `""` | 
| `secret.imageCredentials[].username` | 私有镜像仓库用户名 | `""` |
| `secret.imageCredentials[].password`| 私有镜像仓库密码 | `""` |
| `storageClassName` |  | `local-path` |
| **fogcloud** | api服务相关配置 | |
| `fogcloud.restartPolicy` | pod重启策略：`Always` | `Always` |
| `fogcloud.image` | 镜像地址 | `ghcr.io/fogcloud-io/fogcloud` | 
| `fogcloud.imageTag` | 镜像版本 | `v4.13.0` |
| `fogcloud.replicas` | deployment复制节点数量 | `3` |
| `fogcloud.strategy.type` | 应用更新策略：`RollingUpdate`，`Recreate`；1）`RollingUpdate`滚动更新；2）`Recreate`重启更新 | `RollingUpdate` |
| `fogcloud.strategy.rollingUpdate.maxSurge` | 应用更新时最大新版本pod新增数量比例| `50%` |
| `fogcloud.strategy.rollingUpdate.maxUnavailable` | 应用更新时的最大不可用pod数量 | `0` |
| **faasbuilder** | | |
| `faasbuilder.createDockerconfigWithFile` | 使用文件创建dockerconfig对象 | `false` |
| **mqttBroker** | | |
| `mqttBroker.enabled` | 是否使用k8s创建mqtt broker | `true` |
| `mqttBroker.nodeSelector.enabled` | 是否启用pod节点选择 | `false` |
| `mqttBroker.nodeSelector.key` | k8s节点名 | |   
| `mqttBroker.tls.enabled` | mqtt应用是否启用tls  | |
| `mqttBroker.tls.createWithCertFile` | 是否使用证书文件创建mqtt应用的sercret对象，启用mqttBroker.tls时有效；若为true，可将*.crt（证书）, *.key（密钥）文件放到fogcloud-charts/configs/cert/mqtt目录下 |  | 
| **rabbitmq** | | |
| `rabbitmq.enabled `| 是否使用k8s创建rabbitmq |  |
| `rabbitmq.tls.enabled` | rabbitmq是否启用tls |  |
| `rabbitmq.tls.createWithCertFile` | 是否使用证书文件创建rabbitmq的sercret对象，启用rabbitmq.tls时有效；若为true，可将*.crt（证书）, *.key（密钥）文件放到fogcloud-charts/configs/cert/amqp目录下 |  | 
| **postgres** | | |
| `postgres.enabled` | 是否使用k8s创建postgresql应用（生产环境不建议使用容器部署） |`true` | 
| **mongodb** | | |
| `mongodb.enabled` | 是否使用k8s创建mongodb应用 （生产环境不建议使用容器部署）| `true`| 
| **redis** | | |
| `redis.enabled` | 是否使用k8s创建redis应用（生产环境不建议使用容器部署）|`true` | 
| **etcd** | | |
| `etcd.enabled` | 是否使用k8s创建etcd（生产环境不建议使用容器部署） |`true` | 
| **minio** | | |
| `minio.enabled` | 是否使用k8s创建minio（生产环境不建议使用容器部署） | `true` |




