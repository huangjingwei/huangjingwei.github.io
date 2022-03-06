# Pipenv 的基本使用


{{< image src="/python/pipenv_usage/pipenv.jpg"  >}}


## 1 pipenv 安装

Pipenv是Python项目的依赖管理器。尽管pip可以安装Python包，但仍推荐使用Pipenv，因为它是一种更高级的工具，可简化依赖关系管理的常见使用情况。

使用pip安装：
```shell
$ pip install --user pipenv
```

`--user`代表[用户安装模式]，以防止破坏任何系统范围的包。

如果安装后, shell中没有`pipenv`，则需要将[用户基础目录]的二进制文件目录添加到`PATH`中。

{{< admonition type=tip title="Tip" open=true >}}

在Linux和macOS上，您可以通过运行`python -m site --user-base`找到用户基础目录，然后把`bin`加到目录末尾。比如，上述命令典型地会打印出 `~/.local`（~ 会扩展为您的家目录的局对路径），然后将 `~/.local/bin` 添加到`PATH`中。

在 Windows 上，您通过运行`py -m site --user-site`找到用户基础目录，然后将`site-packages`替换为`Scripts`。比如，上述命令可能返回`C:\Users\Username\AppData\Roaming\Python310\site-packages`，然后您需要在`PATH`中包含`C:\Users\Username\AppData\Roaming\Python310\Scripts`。
{{< /admonition >}}

## 2 创建虚拟环境

### pipenv --python

可以指定版本创建，如果指定版本不存在:

```shell
pienv --python 3.8

Warning: Python 3.8 was not found on your system...
```
### pipenv --three / --two

或者指定`Python 3/2`创建虚拟环境：

```shell
pipenv --three / --two
```

{{< admonition type=note title="Note" open=true >}}
以上i这两种创建指令会自动创建`Pipfile`，但不会有`Pipfile.lock`。
{{< /admonition >}}

### pipenv install

```shell
pipenv install
pipenv install --three / --two
```

{{< admonition type=note title="Note" open=true >}}
`pipenv install`在一个初始化工程中会自动创建出`Pipfile`和`Pipfile.lock`。

如果在一个已经是pipenv管理的工程中执行该命令，会在本地创建虚拟环境并自动安装`Pipfile`中的包。
{{< /admonition >}}

等价指令：

1.`pipenv install`在新建工程中：

`pipenv install` = `pipenv --three` + `pipenv lock` + `pipenv sync`

2.`pipenv install`在已有`Pipfile`的工程中：

`pipenv install` = `pipenv update` = `pipenv lock` + `pipenv sync`

## 3 Pipfile 和 Pipfile.lock

基于一个空项目创建的`Pipfile`文件内容如下：

```toml
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]

[dev-packages]

[requires]
python_version = "3.10"
```
基于`pipenv install`会多出一个`Pipfile.lock`:

```json
{
    "_meta": {
        "hash": {
            "sha256": "fedbd2ab7afd84cf16f128af0619749267b62277b4cb6989ef16d4bef6e4eef2"
        },
        "pipfile-spec": 6,
        "requires": {
            "python_version": "3.10"
        },
        "sources": [
            {
                "name": "pypi",
                "url": "https://pypi.org/simple",
                "verify_ssl": true
            }
        ]
    },
    "default": {},
    "develop": {}
}
```

安装package：

```shell
pipenv install requests
pipenv install pytest --dev
```

`Pipfile`自动更新：
```toml
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]
requests = "*"

[dev-packages]

[requires]
python_version = "3.10"
```

为限制篇幅，以下的`Pipfile.lock`是只安装`requests`的内容：

```json
{
    "_meta": {
        "hash": {
            "sha256": "a416d48a2c30d4acf425cb96d7ac6672753db8e8f6c962a328848db5b9a290a1"
        },
        "pipfile-spec": 6,
        "requires": {
            "python_version": "3.10"
        },
        "sources": [
            {
                "name": "pypi",
                "url": "https://pypi.org/simple",
                "verify_ssl": true
            }
        ]
    },
    "default": {
        "certifi": {
            "hashes": [
                "sha256:78884e7c1d4b00ce3cea67b44566851c4343c120abd683433ce934a68ea58872",
                "sha256:d62a0163eb4c2344ac042ab2bdf75399a71a2d8c7d47eac2e2ee91b9d6339569"
            ],
            "version": "==2021.10.8"
        },
        "charset-normalizer": {
            "hashes": [
                "sha256:2857e29ff0d34db842cd7ca3230549d1a697f96ee6d3fb071cfa6c7393832597",
                "sha256:6881edbebdb17b39b4eaaa821b438bf6eddffb4468cf344f09f89def34a8b1df"
            ],
            "markers": "python_version >= '3'",
            "version": "==2.0.12"
        },
        "idna": {
            "hashes": [
                "sha256:84d9dd047ffa80596e0f246e2eab0b391788b0503584e8945f2368256d2735ff",
                "sha256:9d643ff0a55b762d5cdb124b8eaa99c66322e2157b69160bc32796e824360e6d"
            ],
            "markers": "python_version >= '3'",
            "version": "==3.3"
        },
        "requests": {
            "hashes": [
                "sha256:68d7c56fd5a8999887728ef304a6d12edc7be74f1cfa47714fc8b414525c9a61",
                "sha256:f22fa1e554c9ddfd16e6e41ac79759e17be9e492b3587efa038054674760e72d"
            ],
            "index": "pypi",
            "version": "==2.27.1"
        },
        "urllib3": {
            "hashes": [
                "sha256:000ca7f471a233c2251c6c7023ee85305721bfdf18621ebff4fd17a8653427ed",
                "sha256:0e7c33d9a63e7ddfcb86780aac87befc2fbddf46c58dbb487e0855f7ceec283c"
            ],
            "markers": "python_version >= '2.7' and python_version not in '3.0, 3.1, 3.2, 3.3, 3.4' and python_version < '4'",
            "version": "==1.26.8"
        }
    },
    "develop": {}
}
```

可以发现：
`Pipfile.lock`会生成所有已下载包的sha256哈希值(包括中间依赖)。这使得pip在不安全网络情况下，保证你安装了你想要的包，或者从一个不信任的PyPI源下载依赖.

查看依赖：
```shell
$ pipenv graph       
pytest==7.0.1
  - atomicwrites [required: >=1.0, installed: 1.4.0]
  - attrs [required: >=19.2.0, installed: 21.4.0]
  - colorama [required: Any, installed: 0.4.4]
  - iniconfig [required: Any, installed: 1.1.1]
  - packaging [required: Any, installed: 21.3]
    - pyparsing [required: >=2.0.2,!=3.0.5, installed: 3.0.7]
  - pluggy [required: >=0.12,<2.0, installed: 1.0.0]
  - py [required: >=1.8.2, installed: 1.11.0]
  - tomli [required: >=1.0.0, installed: 2.0.1]
requests==2.27.1
  - certifi [required: >=2017.4.17, installed: 2021.10.8]
  - charset-normalizer [required: ~=2.0.0, installed: 2.0.12]
  - idna [required: >=2.5,<4, installed: 3.3]
  - urllib3 [required: >=1.21.1,<1.27, installed: 1.26.8]
```

`requests`的依赖有`certifi`、`charset-normalizer`、`idna`、`urllib3`，这些信息同样被记录在`Pipfile.lock`。

## 4 版本管理

官方建议：

> Generally, keep both Pipfile and Pipfile.lock in version control.
> 
> Do not keep Pipfile.lock in version control if multiple versions of Python are being targeted.

参照[#598]。有个观点时`Pipfile.lock`可以精准控制中间依赖的版本，我参与的项目是将`Pipfile.lock`一并纳入版本管理。

## 5 可修改依赖 (如 -e . )

你可以让Pipenv以可修改模式安装某个路径，通常用于开发Python包时，安装当前工作目录。

```shell
$ pipenv install --dev -e .

$ cat Pipfile
[dev-packages]
"e1839a8" = {path = ".", editable = true}
```

所有次级依赖也会加到`Pipfile.lock`中。如果没有加`-e`选项次级依赖将不会加到`Pipfile.lock`中。

## 6 使用pipenv进行部署

可以用pipenv进行部署，在运行环境中安装`Pipfile.lock`中得依赖。

```shell
pipenv install --deploy
pipenv sync
```
相关操作澄清：

- `pipenv install --deploy`是直接通过`Pipfile.lock`安装的，当`Pipfile.lock`的package版本过时或者python版本不对时，会失败。

- `pienv install --deploy`和`pipenv install`的区别在于后者是以`Pipfile`安装的，如果package未指定版本时或者有新的package，会重新lock生成新的`Pipfile.lock`。

- `pipenv sync`是直接根据`Pipfile.lock`中的依赖版本准确安装。

- `pipenv install --ignore-pipfile`类似`pipenv sync`有点类似，只依赖`Pipfile.lock`。但是前者会re-lock，如`Pipfile.lock`中的package版本过时，会被更新。

## 7 Pipfile vs setup.py

详见：[Pipfile vs setup.py]。

可以通过以下命令将`setup.py`中的相关依赖安装到你的虚拟环境和`Pipfile`中。

```shell
pipenv install -e .
```


[用户安装模式]:https://pip.pypa.io/en/stable/user_guide/#user-installs
[用户基础目录]:https://docs.python.org/3/library/site.html#site.USER_BASE
[#598]: https://github.com/pypa/pipenv/issues/598
[Pipfile vs setup.py]:https://pipenv-zh.readthedocs.io/zh_CN/latest/advanced.html#pipfile-vs-setup-py


