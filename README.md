# v2ray-padavan-doh

本文为在 RM2100 路由器使用 padavan(N56U/改华硕) 固件安装配置 v2ray/xray 的进阶流程，v2ray/xray 一揽子方案（简单）在这里[v2ray-padavan](https://github.com/felix-fly/v2ray-padavan)，本文同时也是建立在此基础上的，有些内容不做过多说明，不明白可以先看一下此文。

笔者编译的固件为纯净版，release 下可自取，只包含了 smartdns 和 v2ray/xray。

前期 v2ray/xray 是手动安装在 storage 下，现在改为将 v2ray/xray 内置到 padavan 固件中。使用 actions 来构建，笔者使用的是 RM2100，如果是 padavan 支持的其它型号的路由，可以参考修改打造你自己的固件。

编译好的纯净版固件可以[在 release 下载](https://github.com/felix-fly/v2ray-padavan-doh/releases)，只包含了 smartdns 和 [v2ray](https://github.com/felix-fly/v2ray-openwrt/releases) / [xray](https://github.com/felix-fly/xray-openwrt/releases)。如需其它插件，可以自行修改 **RM2100.config** 文件进行编译。

其中 RM2100_X.trx 是我之前自己用的，固件只包含了 smartdns，可以配合独立的 v2ray/xray 设备使用（类似旁路由）。

**个人目前在用 RM2100_xray.trx，很稳定，性能也很强大，有条件的可以挑战一下科学500兆：**

* [榨干 MT7621 极限性能，科学跑满500兆有木有可能](https://itcao.com/2021/08232231.html)

## 证书

v2ray/xray 使用 tls 相关协议时会需要验证证书的有效性，而 padavan 默认是没有包含 ssl 根证书的。简单的方式可以配置 v2ray/xray 不验证，但是出于安全性的考虑，还是要验证的比较好。鉴于此，固件里内置了一份证书合集，放在了

* ```/usr/lib/cacert.pem```

证书文件来自于 **https://curl.haxx.se/docs/caextract.html**

## 配置 xray **\*\*推荐\*\*** （二选一）

将 xray 文件夹里的所有文件上传至路由器 /etc/storage/xray 目录，并添加脚本执行权限

```bash
chmod +x /etc/storage/xray/*.sh
```

## 配置 x2ray （二选一）

将 v2ray 文件夹里的所有文件上传至路由器 /etc/storage/v2ray 目录，并添加脚本执行权限

```bash
chmod +x /etc/storage/v2ray/*.sh
```

## dnsmasq配置（二选一：不使用smartdns）

* 将 dnsmasq 文件夹里的所有文件上传至路由器 /etc/storage/dnsmasq 目录

* 通过padavan管理界面修改，dnsmasq拦截广告并将gw域名转给v2ray进行解析。

**内部网络(LAN) -> DHCP服务器 -> 自定义配置文件 "dnsmasq.conf"**

下面两种配置方式，选择其中一种即可

### 方式一：通过 gfwlist 和 addn-hosts 参数配置

这种方式通过 addn-hosts 方式屏蔽广告域名解析，测试下来有问题，直接 ping 域名会返回 0.0.0.0 但是通过 curl 或者浏览器访问域名会正常，此时再 ping 就会得到正确的地址，过一会再 ping 又回到 0.0.0.0 具体什么原因造成的目前还不清楚。如果你知道，请一定要告诉我。据说这种 addn-hosts 方式在加载大量域名时查询性能更好。

```bash
addn-hosts=/etc/storage/dnsmasq/ad.list
gfwlist=/etc/storage/dnsmasq/gw.list@127.0.0.1#1053^gw
```

### 方式二：加载自定义配置文件

目前广告列表已经超过五万条了，dnsmasq 加载后查询效率已经有所下降，具体表现为 cpu 使用率较之前有升高。如果影响正常使用，可以不加载广告规则，或者选用下面 smartdns 的方案。

```bash
conf-dir=/etc/storage/dnsmasq, *.hosts
```

**高级设置 -> 自定义设置 -> 脚本 -> 在防火墙规则启动后执行:**

```bash
/etc/storage/dnsmasq/iptables.sh
```

## dnsmasq配置（二选一：使用smartdns提供dns解析服务）

[**具体配置参考关于smartdns的描述**](./smartdns.md)

## 设置 v2ray/xray 开机自动启动

**系统管理 - 服务: 调度任务 (Crontab): 添加一行**

```bash
*/5 * * * * /etc/storage/xray/check.sh > /dev/null
```

或者

```bash
*/5 * * * * /etc/storage/v2ray/check.sh > /dev/null
```

## 保存软件及配置

padavan系统文件系统是构建在内存中的，重启后软件及配置会丢失，所以操作完成后，需要将相关配置写入闪存。

**高级设置 -> 系统管理 -> 配置管理 -> 保存内部存储到闪存: 提交**

由于配置文件比较大，提交保存操作需要耐心等待几秒钟，以确保写入成功。

如果一切顺利，重启路由hi起来。Good luck!

## 更新记录
2021-09-08
* 推荐使用smartdns及xray
* iptables使用tproxy方式

2021-05-10
* 增加xray
* 使用自行动态编译smartdns

2021-03-08
* 增加smartdns

2020-11-13
* 内置证书文件
* 增加ad-ext，过滤更为严格

2020-07-16
* 内置v2ray
* 去掉doh相关，dns通过v2ray解析

2019-08-06
* 草稿，doh待处理
