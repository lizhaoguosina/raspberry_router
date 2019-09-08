# 树莓派路由器计划

## 将树莓派改装成一个路由器，并安装ntp服务作为局域网授时服务器

## 有可能将会安装pi-hole作为拦截广告使用……不过不一定按期更新，复习炸了才会更新…… 

***

>参考了 https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md 和 https://vitux.com/how-to-install-ntp-server-and-client-on-ubuntu/ 两篇文章

>随后将会安装wordpress等搭建一个局域网智能网关……
---
### 在 raspbian-2019-07上验证通过，ssh可以连接就可以使用

1. 更新与安装必要的包
```shell
sudo apt update
sudo apt -y upgrade
sudo apt -y install dnsmasq hostapd
sudo systemctl stop dnsmasq
sudo systemctl stop hostapd
```
使用nano作为编辑器，如果你擅长使用vi/vim，编辑内容是相同的。此处我们需要安装dnsmasq 和 hostapd 两个包，并且先暂停服务以便于操作接下来的内容。

2. Configuring a static IP
>给树莓派的无线网卡设置固定IP方便作为路由使用。 

```shell
sudo nano /etc/dhcpcd.conf
```
>在文件末尾编辑，设定如下所示
```shell
interface wlan0
    static ip_address=192.168.4.1/24
    nohook wpa_supplicant
```
此处填写的ip_address只要符合网络要求即可，通常我们设置为"192.168.x.x"，也可以设置非24子网掩码，只要符合要求即可。

**有些教程可能要求你修改 */etc/network/interfaces* 这个文件，但这个文件已经被废除。所以你应该修改的是 */etc/dhcpcd.conf***

>重启**dhcpcd**服务使得新的wlan0配置生效

```shell
sudo service dhcpcd restart
```

3. 设置DHCP服务器

把旧的配置文件重命名并新建一个来编辑

```shell
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
sudo nano /etc/dnsmasq.conf
```

在文件末尾添加

```shell
interface=wlan0      
dhcp-range=192.168.4.2,192.168.4.249,255.255.255.0,24h
```

在设置wlan0的固定ip时，我们设置的是 *"192.168.4.1/24"* 也就是说我们的DHCP 服务器ip是 *"192.168.4.1"* 所以我们的DHCP池配置成为 *"192.168.4.2,192.168.4.249"* ，即DHCP池范围在 *"192.168.4.2-192.168.4.249"* 。这一步只要配置符合IPv4要求即可。 *"/24"* 等同于 *"255.255.255.0"* 。 *"24h"* 代表单个ip的租约是24h

重启 **dnsmasq** 服务使得DHCP服务生效

```shell
sudo systemctl reload dnsmasq
```

4. 配置接入点的ssid和密码等信息

>编辑 /etc/hostapd/hostapd.conf文件以编辑你的无线接入点信息，通常情况下这应该是一空文件

```shell
sudo nano /etc/hostapd/hostapd.conf
```
改动结果应像下文

```shell
interface=wlan0
# 设置使用那块网卡作为接入点网卡
driver=nl80211
# 使用nl802.11协议跑（通常都这个）
ssid=NameOfNetwork
# 设置ssid信息，就是无线网名称
hw_mode=g
# 802.11不同模式
channel=7
# 不同的信道
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=AardvarkBadgerHedgehog
# 无线密码
wpa_key_mgmt=WPA-PSK
# 加密方式
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

**hw_mode** 选项决定使用不同的频率跑ssid，具体配置见下

 * a = IEEE 802.11a (5 GHz)
 * b = IEEE 802.11b (2.4 GHz)
 * g = IEEE 802.11g (2.4 GHz)
 * ad = IEEE 802.11ad (60 GHz)

向系统声明配置文件于何处
```shell
sudo nano /etc/default/hostapd
```

找到 **#DAEMON_CONF** 这行，并编辑如下
```shell
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```
5. 启动ap
启动hostapd、dnsmasq服务
```shell
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
sudo systemctl start hostapd
sudo systemctl restart dnsmasq
```
检查两个服务是否已经正常运行。不正常需要寻找问题所在

```shell
sudo systemctl status hostapd
sudo systemctl status dnsmasq
```

6. 添加转发

***如果在接下来出现报错，通常你需要重启后再进行下一步……原本教程中并没有这行……***
```shell
sudo reboot
```
编辑 **/etc/sysctl.conf** 取消下面这行的注释（也可以自己敲进去）
```shell
net.ipv4.ip_forward=1
```
添加流量伪装……（翻译很操蛋但大致就是告诉流量包下一步怎么走）
```shell
sudo iptables -t nat -A  POSTROUTING -o eth0 -j MASQUERADE
```
保存IP表
```shell
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
```
编辑 /etc/rc.local使得开机就可以启用转发 
```shell
sudo nano /etc/rc.local
```
在"exit 0"上添加这句 
```shell
iptables-restore < /etc/iptables.ipv4.nat
```
重启以确保可以运行……此时你应该已经多了一个无线信号可以使用了

接下来安装ntp服务

7. 升级更新列表和包
```shell
sudo apt update && sudo apt -y up grade
```
8. 安装ntp server
```shell
sudo apt install ntp
```
9. 选一个离你最近的NTPPool（距离会影响精度……）

```shell
sudo nano /etc/ntp.conf
```
在这里你可以找到大多数的NTPPool 
>https://support.ntp.org/bin/view/Servers/NTPPoolServers

10. 重启ntp服务并检查是否正常运行
```shell
sudo service ntp restart
sudo service ntp status
```
---
以下是吐槽
###### 哎我也是真他妈闲的蛋疼才来搞这个，明明考研都复习不完了还在玩这个……没救了没救了

---
