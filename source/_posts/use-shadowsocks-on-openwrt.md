---
layout: post
title: 在 OpenWRT 上使用 ShadowSocks 建立透明代理
date: 2015-02-10
---

## 0.前言


因为众所周知的原因，使用 ShadowSocks 对网络进行加速已经是一种普遍使用的行为。但是不管是 VPN 也好，ShadowSocks 也罢，在使用中总会碰见各种不完美的问题。如每个终端都需要单独配置，连接上服务器后所有流量都会走该服务器等等。撰写本文的目的是对最近在路由器上解决这些问题的一个总结，如果在路由器上部署好了 ShadowSocks 的话，不但所有接入该路由器的设备能自动走 ShadowSocks ，而且可以针对网站选择合适的线路，这些工作对客户端来说都是透明的，只要跟平常一样连接上路由器就好。

## 1.准备工作

### 硬件准备：
可以刷 OpenWRT 的路由器一台（废旧 PC 也可以），建议使用小米路由器mini或联想的New Wifi Mini，两者硬件配置和价格几乎都一样，都是便宜耐操的货（认为本文是软文的话现在就可以 Ctrl + W 了）。

### 软件准备：
路由器对应的 OpenWRT 固件。下载地址 [点击这里](http://downloads.openwrt.org.cn/PandoraBox/)

shadowsocks-libev-spec 的 ipk 包，项目地址 [点击这里](http://sourceforge.net/projects/openwrt-dist/files/shadowsocks-libev/) 请下载 [luci-app-shadowsocks](http://sourceforge.net/projects/openwrt-dist/files/luci-app/shadowsocks-spec/) 以及对应 CPU 架构的安装包。 如果是刚才提到的那两个型号，请下载 ramips 架构最新的 shadowsocks-libev-spec。

几个版本的区别是，带 spec 的是针对 OpenWRT 优化过的版本，只要装上了对应的 luci-app 就可以通过 Web 界面管理。而名字带 polarssl 的使用的是 polarssl 库，据说相当于名字没带 polarssl 的版本来说，体积会更小。但是默认的使用 openssl 的版本性能会更好。

## 2.操作步骤

### 2.1 刷入 OpenWRT
以下以小米路由器mini为例子，假定本文的读者已打开了 SSH 登录（详细操作请查询官方的操作教程。注意：获得 SSH 会失去官方保修，本文不对这种行为负任何责任）。

我这里使用的是小米路由器mini的 [PandoraBox r355固件](http://downloads.openwrt.org.cn/PandoraBox/Xiaomi-Mini-R1CM/stable/PandoraBox-ralink-xiaomi-mini-r355-20150114.bin)，把该固件上传到路由器的 /tmp 目录下后，SSH 登录上去并执行以下指令：

```bash
mtd -r write /tmp/PandoraBox-ralink-xiaomi-mini-r466-20140701.bin firmware 
```

执行后路由器会自动重启，重启完成后即可通过 WiFi 或网线连接到已经刷入 OpenWRT 的路由器。

如果是使用 x86 的 PC 作为软路由的话，把U盘/存储卡做成 OpenWRT 的启动盘然后重启即可。

### 2.2 修改 opkg.conf 使包管理器生效
因为这个版本的 opkg.conf 有错，导致 opkg 有问题，所以请在 Web 管理界面中的”系统“→”软件包“→”管理“中，把几个镜像源改成这两个地址：

```
src/gz oldpackages http://downloads.openwrt.org/barrier_breaker/14.07/ramips/mt7620a/packages/oldpackages 
src/gz packages http://downloads.openwrt.org.cn/PandoraBox/ralink/mt7620/packages
```

另外因为架构没填写正确，部分 ipk 软件包（如 ShadowSocks）会安装失败，所以还得继续修改 opkg.conf 加入以下几句：

```
arch all 100
arch noarch 200
arch ralink 300
arch ramips_24kec 400
```

### 2.3 打开 SSH
这个版本默认不带 SSH，请在 Web 界面的“系统”→”软件包“中安装 openssh-server。安装完成后即可通过 SSH 登录到路由器。

Windows 下可以使用 putty 作为 SSH 客户端登录

### 2.4 ShadowSocks 的安装及配置

用 WinSCP 工具上传 shadowsocks-libev-spec_2.1.3-1_ramips_24kec.ipk 以及 luci-app-shadowsocks-spec_1.3.0-1_all.ipk 到路由器的 /tmp 下，并在 SSH 窗口里执行：

```
cd /tmp
opkg install shadowsocks-libev-spec_2.1.3-1_ramips_24kec.ipk
opkg install luci-app-shadowsocks-spec_1.3.0-1_all.ipk
```

即可完成 ShadowSocks 部分的配置。

然后登录到路由器的 Web 管理界面，“服务”→“Shadowsocks”，把“使用配置文件”的勾去掉，填入你的 ShadowSocks 服务器信息，保存并应用。

然后在 SSH 窗口里执行：

```
/etc/init.d/shadowsocks enable
/etc/init.d/shadowsocks start
```

刷新 Web 的 Shadowsocks 管理界面，如果显示“Shadowsocks 已在运行”即已成功启动。 

### 2.5 DNS的配置
在进行以下操作前，请先在 Web 管理界面的“网络”→“DHCP/DNS”→“HOSTS和解析文件”中勾上“忽略解析文件”以防止上级 DNS 的污染。

这个版本的 Shadowsocks 可以打开端口转发，如启用，默认会把本地的 5300 端口转发到 8.8.4.4#53 端口。所以如果只需要使用 Google 的 DNS 服务器作为上游的话，只需要在/etc/dnsmasq.conf 的末尾里添加一行：

```
server=127.0.0.1#5300
```

但是如果全走  Google 的公共 DNS 的话，虽然走加密通道可以防止 DNS 污染，但是同样的，对有 CDN 优化的网站来说，返回的将会是国外域名。如淘宝就会打开国际版。所以必须要加入针对国内网站的优化。

首先我们继续在 /etc/dnsmasq.conf 的末尾里继续加入：

```
conf-dir=/etc/dnsmasq.d
```

然后在 SSH 窗口中输入以下命令：

```
# 创建 dnsmasq 的配置目录
mkdir /etc/dnsmasq.d
# 安装 git
opkg install git
# 克隆 dnsmasq-china-list 项目到本地
git clone git://github.com/felixonmars/dnsmasq-china-list /root/dnsmasq-china-list 
# 软连接配置文件到 dnsmasq 的配置目录
ln -s /root/dnsmasq-china-list/accelerated-domains.china.conf /etc/dnsmasq.d/accelerated-domains.china.conf
# 重启dnsmasq 服务
/etc/init.d/dnsmasq restart
```

另外有一点要注意的是，如果 ShadowSocks 的服务器填写的是域名而不是 ip 的话，会陷入一个死循环（ShadowSocks 未启动→域名无法解析→无法启动 ShadowSocks 服务），所以对于这种情况，在重启 dnsmasq 服务前，建议在 /etc/dnsmasq.d/ 下新建一个 ss-server.conf，内容为：

```
server=/服务器的根域名xxx.com/114.114.114.114
```

或者在 /etc/hosts 里写死该服务器对应的 ip 地址。

### 2.6 进阶：自动更新配置文件

因为安装包的问题所以这个 git 无法使用 pull 以及 merge 命令，我们首先对其修复。

在 SSH 窗口中输入以下命令：

```sh
ln -s /usr/bin/git /usr/bin/git-merge
ln-s /usr/bin/git /usr/bin/git-pull
```

然后新建 /root/update.sh，内容如下：

```
wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > /etc/shadowsocks/ignore.list
cd /root/dnsmasq-china-list
git pull --rebase

/etc/init.d/dnsmasq restart
/etc/init.d/shadowsocks restart
```

然后用 chomod +x /root/update.sh 命令给予该脚本执行权限，然后执行 /root/update.sh 试试看。正常的话配置文件应该会成功更新。

在路由器的 Web 管理界面中点击“系统”→“计划任务”中，输入以下内容：

```
30 4 * * * /root/update.sh
```

保存后，应该每天的凌晨4点就会自动执行这个更新脚本了。

## 3.小结

经过以上的调教后路由器应该就能变身智能分流的路由器了。如果在测试中发现什么问题，欢迎留言讨论。