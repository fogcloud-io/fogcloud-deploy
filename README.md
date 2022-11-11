# fogcloud部署

## foglcoud相关服务镜像

```
ghcr.io/fogcloud-io/fogcloud:latest
ghcr.io/fogcloud-io/fogcloud-web:latest
ghcr.io/fogcloud-io/faas-builder:latest
ghcr.io/fogcloud-io/fogcron-scheduler:latest
```

## 配置要求
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

## 3 使用helm部署




