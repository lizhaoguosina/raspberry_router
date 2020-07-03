# 树莓派进行bt/pt下指站教程

## 在树莓派或者其他基于debian的Linux系统中进行bt/pt下载，后期会新增docker版。

***

>参考文章已经不可考，但还记得一部分

***

### 通过命令行进行安装，并安装更好看的页面

**1.** 更新与安装transmission

```shell
sudo apt update 
sudo apt upgrade -y
sudo apt install transmission-daemon -y
```

其实这里不需要你再折腾什么了，service也不需要终止后再编辑，直接编辑就好了。毕竟Linux的思想是，“万物皆文件”。

**2.** 对于必要的配置文件进行编辑

这里主要是针对配置文件进行修改，安装完更好的web界面以后可以进行GUI的修改。在后文中会提到.

transmission-daemon的配置文件是/etc/transmission-daemon/settings.json 这里我选择我最熟悉的nano进行编辑，也可以根据需要选择其他编辑器。

```shell
sudo nano /etc/transmission-daemon/settings.json
```

在配置文件中，重点在于以下几个配置

**(1**
```shell
"download-dir": "/home/wwwroot"
```
这个指的是你的默认下载目录。当然也可以在后面安装了新的页面控制后更改。需要注意的是，此处设置的文件夹最好是所有用户均有读写权限为宜。更改方式
```shell
sudo chmod 766 /home/wwwroot
```

**(2.**
```shell
"rpc-password":{}
```
这个是你的web控制端的登陆密码，输入明文就好，服务起来的时候会自动加密的

**(3.**
```shell
"rpc-whitelist": "0.0.0.0",
"rpc-whitelist-enabled": false,
```
这两行建议改成这样的，因为默认是禁止远程访问的

**(4.**
```shell
"rpc-username": ""
```
引号内填写登陆时的用户名

**(5.**
```shell
"rpc-port": 9091,
```
这个**9091**指的是访问时的端口号，你也可以根据自己喜好更改成其他的端口号，但务必保证已经将此端口放行。在安装完transmission之后，系统默认放行了9091，因此我不太建议自行更改此参数。

修改完毕后保存并退出。使用以下命令重新加载配置文件和重启服务(注：此命令可能在之后的版本中移除，可使用较新版本命令所替代)
```shell
sudo service transmission-daemon reload
sudo service transmission-daemon restart
```
**3.** 安装更好看的web-control界面

此处使用的是**github** 的一个开源项目 
>https://github.com/ronggang/transmission-web-control
按照其说明下载安装即可。