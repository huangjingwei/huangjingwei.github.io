# 使用hugo镜像

{{< image src="/Hugo/image/klakegg-hugo.png"  >}}

`klakegg/hugo`是官方推荐的，包含hugo开源静态网站生成器的镜像。
<!--more-->

## docker-compose

```yml
version: '3'
services:
  hugo:
    container_name: "×××.github.io"
    image: "klakegg/hugo:latest"
    ports:
      - "1313:1313"
    volumes:
      - "~/×××.github.io.source:/×××.github.io.source"
    working_dir: "/×××.github.io.source"
    command: ["server", "-D"]

```

## 使用hugo

启动服务：
```shell
docker-compose up -d
```

启动服务之后就会有相关容器在运行。

如果要新建博文：

```shell
docker exec -it ${container_id} hugo new content/posts/Ubuntu/test.md
```

以上面为例，可以调用hugo的其他命令。


