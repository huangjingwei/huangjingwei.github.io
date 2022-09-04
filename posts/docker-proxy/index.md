# Docker 安装和网络代理


## 1 Docker

Docker是一个开放平台，可用于容器镜像的开发、交付和运行。

Docker名词一般是泛指。一般会代指Docker引擎(Docker Engine)或Docker注册中心(Docker registry)。

我们这边讨论的是Docker引擎。其主要有3部分组成：

- Docker守护进程(Docker daemon / dockerd)。
- Docker Engine API。
- Docker 客户端。

## 2 Docker的安装卸载

Docker的版本自`17.03`后分为CE(Community Edition)和EE(Enterprise Edition)，个人使用安装CE版本。

### 推荐安装方法

Linux的发行版一般会自带docker的软件包或者下载`.deb`文件手动安装，
但是本文推荐的安装方法是通过apt包管理工具安装，参照官网[安装向导]。

安装之前建议先卸载旧版本：

```Bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

添加使用HTTPS传输的软件包和CA证书：

```Bash
$ sudo apt-get update
$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

添加软件源需要的GPG密钥：

```Bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# 如果是国内源：
$ curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```

配置仓库：

```Bash
$ sudo add-apt-repository \
    "deb [arch=$(dpkg --print-architecture) https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable"
```

或者直接在`/etc/apt/sources.list.d/`创建一个`docker.list`文件,添加上式中的引号中的内容，
并修改内容中修改架构和发行版名称。

执行安装：

```Bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

建立docker用户组：

```Bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

如果一切顺利的话，就安装成功了。

### 卸载方法

卸载Docker Engine：

```Bash
sudo apt-get purge -y docker-engine docker docker-ce docker-ce-cli containerd.io
sudo apt-get autoremove -y --purge docker-engine docker docker-ce docker-ce-cli containerd.io
```

以上的命令不会删除主机上的镜像、容器、卷和用户创建的配置文件等，如需清理：

```Bash
sudo rm -rf /var/lib/docker /var/lib/containerd /etc/docker
sudo rm /etc/apparmor.d/docker
sudo groupdel docker
sudo rm -rf /var/run/docker.sock
```

## 3 网络代理配置

在公司经常需要挂代理才可以正常访问互联网。如果是这种情况，需要为Docker配置代理。
在Docker的使用中，需要访问外网的场景一般有：

- dockerd代理配置
- Container代理配置
- docker build代理配置

下文主要参考：[Docker的三种网络代理配置]。

### dockerd代理配置

当你执行`docker login`、`docker push`、`docker pull`时，实际上是在和docker-client交互，
这时候docker-client操作dockerd去访问外网的镜像仓库时，就会有相关的API发出，这时候需要给dockerd去配置代理。
docker受到systemd的管控，所以需要修改的是systemd的配置。

```Bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo touch /etc/systemd/system/docker.service.d/proxy.conf
```

这里新创建的配置文件的命名只要符合`*.conf`的形式即可。添加内容如下：

```sh
[Service]
Environment="HTTP_PROXY=http://<proxy-addr>:<proxy-port>"
Environment="HTTPS_PROXY=http://<proxy-addr>:<proxy-port>"
Environment="NO_PROXY=localhost,127.0.0.1,docker-registry.somecorporation.com"
```

其中的代理要换成可用的免密代理。

{{< admonition type=tip title="生效条件" >}}
需要重启dockerd才可生效。

重启指令：

```sh
sudo systemctl daemon-reload
sudo systemctl restart docker
```

{{< /admonition >}}

### Container代理配置

#### 配置文件

当容器运行时，容器内的运用需要代理访问时，需要配置`~/.docker/config.json`，当然只生效在Docker的`17.07`及其以后版本。

```yaml
{
 "proxies":
 {
   "default":
   {
     "httpProxy": "http://<proxy-addr>:<proxy-port>",
     "httpsProxy": "http://<proxy-addr>:<proxy-port>",
     "noProxy": "localhost,127.0.0.1,docker-registry.somecorporation.com"
   }
 }
}
```

以上的配置可以一劳永逸。

#### 环境变量

当然，还有一种是在运行容器时通过环境变量的形式注入，命令是`-e`或者`--env`。

{{< admonition >}}
注入的代理根据真实需要，如容器是基于ubuntu系统的应该配置`http_proxy`、`https_proxy`、`no_proxy`等。
{{< /admonition >}}

{{< admonition type=tip title="生效条件" >}}
对已经启动的容器无效，对配置后启动的容器立即生效。
{{< /admonition >}}

### docker build代理配置

通过`docker build`构建镜像，是启动容器并逐层构建。但是上文中的配置文件对其无效。

Dockerfile也可以设置环境变量，有`ENV`和`ARG`，这两者的作用一致，但是通过`ENV`设置的环境变量会一并带到生成的镜像中。
`ARG`则不会，只在构建的阶段有效。

Dockerfile的`ARG`和`docker build`的`--build-arg`是一致的，
建议在构建时通过`--build-arg <varname>=<value>`参数来指定或重设置这些变量的值。

所以，`docker build`配置代理：

```Bash
docker build . \
    --build-arg "http_proxy=http://<proxy-addr>:<proxy-port>" \
    --build-arg "https_proxy=http://<proxy-addr>:<proxy-port>" \
    --build-arg "no_proxy=localhost,127.0.0.1,docker-registry.somecorporation.com" \
    -t your/image:tag
```

同样，由于是通过环境变量来配置代理，配置生效的环境变量应该要符合实际镜像的要求。

{{< admonition >}}
如果代理使用的是`localhost:3128`这类，必须加上`--network host`代理才能生效。
或者直接配置代理的外部IP。
{{< /admonition >}}

{{< admonition type=tip title="生效条件" >}}
执行`docker build`构建镜像时立即生效。
{{< /admonition >}}

[安装向导]:https://docs.docker.com/engine/install/ubuntu/
[Docker的三种网络代理配置]:https://note.qidong.name/2020/05/docker-proxy/

