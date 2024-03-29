# 花生壳内网穿透


{{< image src="/Ubuntu/phddns/limit.png"  >}}
使用花生壳，实现内网穿透，以便在公网的机器可以正确路由到内网的主机上的服务。
<!--more-->

## 安装和卸载

```shell
S wget "https://down.oray.com/hsk/linux/phddns_5.2.0_amd64.deb" -O phddns_5.2.0_amd64.deb
$ sudo dpkg -i phddns_5.2.0_amd64.deb #安装
$ sudo dpkg -r phddns  #卸载
```

## 运行

```shell
$ phddns
Phtunnel Serive called with  unknown argument
(phddns  |start|status|stop|restart|reset|enable|disable|version)


$ phddns status
 +--------------------------------------------------+
 |          Oray PeanutHull Linux 5.2.0             |
 +--------------------------------------------------+
 |              Runstatus: ONLINE                   |
 +--------------------------------------------------+
 |              SN: oray************                |
 +--------------------------------------------------+
 |    Remote Management Address http://b.oray.com   |
 +--------------------------------------------------+
```

## SN码激活

`phddns status`会提示有远程的管理地址：`http://b.oray.com`,选择`SN登录`。
帐号在`phddns status`指令返回，登录的初始密码为`admin`。

首次登录，需先激活。所以需要注册贝锐的管理帐号用于以上SN码激活，注册地址：[注册帐号]。

激活成功后，再用SN码登录时，密码自动更改为贝锐的管理帐号的密码。

## TCP添加映射

进入花生壳管理平台。若绑定SN码的帐号只有动态域名解析功能，需使用内网穿透功能时，可点击“免费开通”，或直接将帐号升级到带内网穿透功能的服务版本。

{{< image src="/Ubuntu/phddns/free-usage.png"  >}}

添加内网穿透映射时，点击页面上的`增加映射`按钮。

根据页面提示填写映射所需的信息，这里以映射Ubuntu系统的SSH服务（22端口）为例：

①应用名称：自定义

②应用图标：自行选择

③映射类型：选择TCP

④映射模板：暂不选择模板

⑤外网域名：选择用作外网访问的域名，域名是平台默认生成，这时候应该有个默认域名可供选择。

⑥外网端口：选择动态端口

⑦内网主机：映射的Ubuntu系统内网IP地址

⑧内网端口：映射的服务类型对应端口22

⑨带宽：购买映射带宽后，可支持给映射分配额外带宽，这里保存默认。

确认映射内容无误后，点击“确定”。

{{< image src="/Ubuntu/phddns/port-mapping.png"  >}}

{{< admonition tip "Tip" true >}}
查看当前的ubuntu是否安装了ssh-server服务。默认只安装ssh-client服务。

```shell
dpkg -l | grep ssh
```

如果没有`openssh-server`，安装：

```shell
sudo apt-get install openssh-server
```

{{< /admonition >}}

这样在外网的电脑上，打开连接SSH服务的工具程序，输入域名与外网端口号就可以访问了。

## HTTP添加映射

HTTP映射和TCP映射大同小异，只不过要向平台缴费`￥6`，然后等平台给一个新的域名。
建议端口号用默认的就可以，然后内网ip和端口按实际的填写。可以开启限制访问。

{{< image src="/Ubuntu/phddns/oder.png"  >}}

## 远程桌面

以Windows远程Ubuntu桌面为例。

Windows的mstsc远程桌面支持远程桌面协议 (RDP)，
Xrdp是一个开源工具，允许用户通过Windows RDP访问Linux远程桌面。

Ubuntu按照xrdp：

```sh
sudo apt install xrdp
sudo systemctl enable xrdp
sudo adduser xrdp ssl-cert
sudo systrmctl restart xrdp
```

xrdp默认使用`/etc/ssl/private/ssl-cert-snakeoil.key`，该文件`ssl-cert`组里的用户是只读的。

{{< admonition tip "Tip" true >}}
在远程桌面前，最好退出其他有在登录的用户，避免黑屏。
{{< /admonition >}}

```sh
$ sudo systemctl status xrdp
● xrdp.service - xrdp daemon
     Loaded: loaded (/lib/systemd/system/xrdp.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-08-31 22:47:17 CST; 2s ago
       Docs: man:xrdp(8)
             man:xrdp.ini(5)
    Process: 17618 ExecStartPre=/bin/sh /usr/share/xrdp/socksetup (code=exited, status=0/SUCCESS)
    Process: 17626 ExecStart=/usr/sbin/xrdp $XRDP_OPTIONS (code=exited, status=0/SUCCESS)
   Main PID: 17627 (xrdp)
      Tasks: 1 (limit: 38360)
     Memory: 872.0K
        CPU: 11ms
     CGroup: /system.slice/xrdp.service
             └─17627 /usr/sbin/xrdp

8月 31 22:47:16 walle systemd[1]: Starting xrdp daemon...
8月 31 22:47:16 walle xrdp[17626]: [INFO ] address [0.0.0.0] port [3389] mode 1
8月 31 22:47:16 walle xrdp[17626]: [INFO ] listening to port 3389 on 0.0.0.0
8月 31 22:47:16 walle xrdp[17626]: [INFO ] xrdp_listen_pp done
8月 31 22:47:16 walle systemd[1]: xrdp.service: Can't open PID file /run/xrdp/xrdp.pid (yet?) after start: Operation not permitted
8月 31 22:47:17 walle systemd[1]: Started xrdp daemon.
8月 31 22:47:18 walle xrdp[17627]: [INFO ] starting xrdp with pid 17627
8月 31 22:47:18 walle xrdp[17627]: [INFO ] address [0.0.0.0] port [3389] mode 1
8月 31 22:47:18 walle xrdp[17627]: [INFO ] listening to port 3389 on 0.0.0.0
8月 31 22:47:18 walle xrdp[17627]: [INFO ] xrdp_listen_pp done
```

由此可以看出，xrdp的默认端口是3389。花生壳上添加tcp的映射方式，内网端口就是xrdp的端口号。

xrdp在局域网中可以延时很低，但是在通过花生壳在公网中访问会非常卡顿。

这里有个临时解决方案：

在Windows远程桌面时，选择显示选项：

- 体验 -> 选择链接速度来优化性能（调制解调器56Kbps。
- 显示 -> 选择桌面的大小（1024×768。

## 总结

对于个人用户，部署友好，3步创建映射，一键内网穿透无需公网IP，无需搭建专线。

但毕竟是商业平台，最多支持2个映射关系。

如果想添加更多的映射，配置相关安全策略，或者在性能上有需求等都是需要付费的。

尊重产权，适当氪金。

## 附录

[花生壳5.0 for Linux使用教程]

[花生壳5.0 for Linux使用教程]:https://service.oray.com/question/11630.html
[注册帐号]:https://console.oray.com/passport/register.html?fromurl=https%3A%2F%2Fwww.oray.com%2Fannouncements%2Faffiche%2F%3Faid%3D793%253Ficn%253Dupdate%26ici%3Dsl_oray_banner&saSDKMultilink=true

