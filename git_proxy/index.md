# Git 网络代理


## 网络代理

git下载代码的时候可以通过https或ssh。有时候拉取GitHub上的代码因为网络原因出现失败，我们可以通过配置网络代理解决。

二者在提交代码时的认证方式不同：

- https：账号和密码认证。
- ssh：证书认证。


### HTTPS

```sh
git config --global http.proxy "http://127.0.0.1:3128"
git config --global https.proxy "http://127.0.0.1:3128"
```

以上配置生效会在用户目录下的`.gitconfig`文件中体现。

```yml
[user]
        name = huangjingwei
        email = example.com
[http]
        proxy = http://127.0.0.1:3128
[https]
        proxy = http://127.0.0.1:3128

```

### SSH

在用户目录下，新建`.ssh/config`。

```sh
Host github.com
   User git
   IdentityFile "C:\Users\your-username\.ssh\id_rsa"
   ProxyCommand connect.exe -H 127.0.0.1:3128 %h %p
```

