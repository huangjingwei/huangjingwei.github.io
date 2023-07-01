# 开发环境准备


## 编排依赖的服务容器

这里，容器化准备开发所依赖的环境。

下面的示例配置了`redis`，`minio`，`postgres_db`, `registry`的服务。其他的开发环境也可以按需添加。

```yaml
---
version: '3'

services:
  redis: 
    image: redis:6.2.7-alpine3.16
    ports:
      - "127.0.0.1:26379:6379"

  minio:
    image: minio/minio:RELEASE.2023-02-27T18-10-45Z
    command:
      - server
      - /srv/minio
    ports:
      - "29000:9000"
    volumes:
      - minio-data:/srv/minio:rw
    environment:
      MINIO_ACCESS_KEY: "access"
      MINIO_SECRET_KEY: "secret-key"

  db:
    image: postgres:12.11-alpine3.16
    ports:
      - "25432:5432"
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: postgres
      POSTGRES_DB: hjw-dev
      POSTGRES_PASSWORD: password

  registry:
    image: registry:2.8.1
    ports:
      - "25000:5000"
    environment:
      REGISTRY_STORAGE: s3
      REGISTRY_STORAGE_S3_ACCESSKEY: access
      REGISTRY_STORAGE_S3_SECRETKEY: secret-key
      REGISTRY_STORAGE_S3_REGION: local
      REGISTRY_STORAGE_S3_BUCKET: registry
      REGISTRY_STORAGE_S3_REGIONENDPOINT: http://localhost:29000

volumes:
  minio-data:
  db-data:
```

## 开机自启动

为了方便，以上的服务可以配置在开机的时候就绪。

新建并编辑`/etc/systemd/system/docker-compose-dev-app.service`

```toml
[Unit]
Description=Docker Compose Develop Application Service
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/home/hjw/Code/python-example/dev
ExecStart=/usr/bin/docker-compose up -d
ExecStop=/usr/bin/docker-compose down
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
```

以上的`WorkingDirectory`需要配置真实的docker-compose文件的所在目录，**需填写绝对路径**。

然后执行：

```sh
sudo systemctl enable docker-compose-dev-app.service
```

如果失败可以用以下命令查看详细原因。

```sh
sudo systemctl status docker-compose-dev-app.service
```

