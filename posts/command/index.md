# 命令备忘录


## ssh公钥免密登录

```sh
ssh-copy-id -i ~/.ssh/id_rsa.pub 用户名@ip地址
```

以上命令会将本地的ssh公钥拷贝到目标机器的`.ssh`目录下，文件名为`authorized_keys`。
当然如果不知道目标机器密码的话，需要将公钥委托给目标机器的管理员进行添加。

## openssl获取服务端证书

```sh
openssl s_client -connect www.github.com:443
```

