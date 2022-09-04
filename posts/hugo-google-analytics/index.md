# Hugo 配置Google Analytics和Search Console


## Google Analytics

Google分析（Google Analytics）是一个由Google所提供的网站流量统计服务。Google 分析（Analytics）现在是互联网上使用最广泛的网络分析服务。

### 账号注册

在[google Analytics]上完成账号注册。

### 创建网络媒体资源

在【管理】【创建网络媒体资源】下，点击【显示高级选项】。 开启【创建 Universal Analytics 媒体资源】，填写网站网址，选择【仅创建 Universal Analytics 媒体资源】，
完成网络媒体资源创建。

### 获取跟踪信息

获取跟踪信息的导航为：【管理】【跟踪信息】【跟踪代码】。有效跟踪信息如下：

- 跟踪id
- 全局网站代码 (gtag.js)

## hugo配置跟踪信息

需要配置的内容：

- config.toml中配置跟踪id
- 在每个网页的index.html中的head标签里添加全局网站代码（gtag.js）

### 配置跟踪id

在config.toml中找到`google Analytics`，并赋值为跟踪id。

```TOML
googleAnalytics = "UA-XXXXXXXXX-1"
```

其中的`UA-XXXXXXXXX-1`需要替换成对应资源的跟踪id。

### 配置gtag.js

在基础模板页中添加全局网站代码（gtag.js）。配置参考：[Configure Google Analytics]。

在项目根目录下新建`layouts`,在新增一个`_internal/google_analytics_async.html`，内容为全局网站代码（gtag.js）。

hugo规定内间模板的文件目录优先级大于主题目录，这样可以不用去更改主题下的配置模板。

这里已知有两种情况：

- 基础模板通过`baseof.html`配置。
- 基础模板通过`head.html`配置。

如果是通过`baseof.html`配置，需复制主题下的`/themes/xxx/layouts/_default`到根目录下的`layouts`下，不同的主题的`_default`下的文件可能略有不同。

```Bash
+---layouts
|   +---_default
|   |   |   baseof.html
|   |   |   section.html
|   |   |   single.html
|   |   |   single.md
|   |   |   summary.html
|   \---_internal
|           google_analytics_async.html
```

如果是`head.html`配置，`head.html`应该在`/themes/xxx/layouts/partials/`下，所以根目录下的`layouts`文件结果如下;

```Bash
+---layouts
|   +---partials
|   |   |   head.html
|   \---_internal
|           google_analytics_async.html
```

无论是`baseof.html`还是`head.html`，都是在head标签内添加内容：

```html
{{- if not .Site.IsServer }}
   {{ template "_internal/google_analytics_async.html" . }}
{{- end }}
```

验证gtag.js的内容是否成功插入，在项目根目录下执行`hugo`或者`hugo --gc`。会在根目录下会有`public`文件生成。查看文件夹下的`index.html`的head标签内是否成功插入gtag.js的内容。

## 3 Search Console

在[Search Console]添加资源并提交验证。选择【网络前缀】，因为已经配置过`Google Analytics`,所以会自动通过谷歌分析完成所有权验证。

## 4 提交站点地图

在`Search Console`中【站点地图】，添加新的站点地图，hugo会自动生成`sitemap.xml`，直接填写并提交即可。我提交后的状态是无法获取，暂未解决。但是可以先通过网址检查，并请求编入索引。然后通过`site:example.com`可以查询是否被成功索引。

[Google Analytics]: https://analytics.google.com/analytics/web/
[Configure Google Analytics]: https://gohugo.io/templates/internal/#use-the-google-analytics-template
[Search Console]:https://search.google.com/search-console/

