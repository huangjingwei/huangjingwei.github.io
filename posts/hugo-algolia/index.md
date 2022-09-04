# Algolia 站内搜索


## 注册

前往[官方网站]使用GitHub或Google帐号登录。登录完成后根据提示信息填写一些基本的信息即可，注册完成后前往Overview，我们可以发现Algolia会默认给我们生成一个APP。该APP下有`API Keys`信息。

在【`Data sources`】 ->【`Indices`】中新建`index`。

Algolia 为我们提供了三种方式来增加记录：

- 手动添加
- 上传json文件
- API

我们这里使用**API**方式来进行数据的添加。

## API插件

要使用API的方式来添加搜索的数据，我们可以自己根据Algolia提供的API文档进行开发，这也是很容易的，为简单起见，我们这里使用一个`hugo-algolia`的插件来完成我们的数据同步工作。

{{< admonition tip "Tip" true >}}
要安装`hugo-aligolia`我们需要先确保我们已经安装了npm或者yarn包管理工具。
{{< /admonition >}}

安装`hugo-aligolia`：

```sh
npm install hugo-algolia -g
```

安装完成后，在我们hugo生产的静态页面的根目录下面新建一个`config.yaml`的文件(和`config.toml`同级)，然后在`config.yaml`文件中指定Algolia相关的API数据。

```yaml
---
baseurl: "blog.huangjingwei.site"
DefaultContentLanguage: "zh-cn"
hasCJKLanguage: true
languageCode: "zh-cn"
title: "wall-e's craft"
theme: "LoveIt"
metaDataFormat: "yaml"
algolia:
  index: "blog.huangjingwei.site"
  key: "********"
  appID: "********"
---
```

以上配置文件中的`index`，`key`，`appID`的值要和`API Keys`的信息一致。

配置完成以后，在根目录下面执行下面的命令：

```sh
$ hugo-algolia -s
JSON index file was created in public/algolia.json
{ updatedAt: '2018-02-23T02:36:09.480Z', taskID: 249063848950 }
```

然后我们可以看到，上面命令执行完成后会在`public`目录下面生成一个`algolia.json`的文件。在[官方网站]打开`Indices`，可以看到已经有几十条数据了。

如果某篇文章不想被索引的话，我们只需要在文件的最前面设置index参数为false即可，`hugo-algolia`插件在索引的过程中会自动跳过它。

## 站点配置

以`LoveIt`主题为例，在`config.yaml`中修改搜索相关的配置，将搜索引擎配置为`algolia`，并根据实际补充相关配置参数。

```toml
    [languages.zh-cn.params.search]
    enable = true
    # 搜索引擎的类型 ("lunr", "algolia")
    type = "algolia"
    # 文章内容最长索引长度
    contentLength = 4000
    # 搜索框的占位提示语
    placeholder = "只支持首页搜索"
    # 最大结果数目
    maxResultLength = 10
    # 结果内容片段长度
    snippetLength = 50
    # 搜索结果中高亮部分的 HTML 标签
    highlightTag = "em"
    # 是否在搜索索引中使用基于 baseURL 的绝对路径
    absoluteURL = false
    [languages.zh-cn.params.search.algolia]
        index = "blog.huangjingwei.site"
        appID = "********"
        searchKey = ""********"
```

`placeholder = "只支持首页搜索"`，目前未解决。

## 附录

[站内搜索插件]

[官方网站]:https://www.algolia.com/
[站内搜索插件]:https://jimmysong.io/hugo-handbook/steps/searching-plugin.html

