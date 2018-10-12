# Openwrt X86_64
- 该文中所需的ipk包均出 github.com/aa65535
## 准备工作

### 下载Openwrt X86_64 最新版本
- 下载地址：https://downloads.openwrt.org/releases/18.06.1/targets/x86/64/

    该版本为18.06.1版本可在`https://downloads.openwrt.org/releases` 选择最新的版本号。
### 下载 physdiskwrite 磁盘写入软件
- 下载地址 https://m0n0.ch/wall/physdiskwrite.php
- 并将 Openwrt X86_64 的镜像文件和physdiskwrite程序放到同一目录下
- 该软件主要用于将OpenwrtX86_64的`.img`镜像文件写入到磁盘中去

### 准备一个PE系统

## 安装Openwrt X86_64 系统
- 首先将所需要的文件拷入U盘然后进入PE系统
- 将要安装系统的硬盘格式化`！！！！！！！！重要`（不格式化会出现无法写入问题）
- 使用cmd命令进入要 physdiskwrite程序所在的目录

输入命令：
```shell
 physdiskwrite.exe -u Openwrt X86_64.img
```
physdiskwrite.exe：程序名  

-u：硬盘大于800M没有800M可以忽略

Openwrt X86_64.img：系统镜像名称
- 此时会出现选择几号磁盘 0一般为系统 1为移动硬盘

  选择要安装的磁盘编号然后回车选择Y开始安装

  出现 xxx/xxx bytes written in total 这说明写入成功，否则为写入失败。查看硬盘分区是否全部删除格式化！！！！

- 重启电脑Openwrt启动成功

- shell连接路由器输入
  ```shell
  opkg updat
  opkg install luci-i18n-base-zh-cn
  ```
  安装中文界面包


## Openwrt 功能实现

### 实现IPXE网络启动
 - 参考路由器IPXE配置

### 修改客户端IP段
 - 修改lan口中的IP地址为 192.168.x.1
 - 修改lan口中的网关为 192.168.x.1
 - 重启网络服务 

### 使用mwan3实现多线网速叠加
- 参考资料：`http://www.right.com.cn/forum/thread-147109-1-1.html`

    `\Openwrt X86_64完整配置\参考资料\使用MWAN3进行多线叠加详细教程 - OPENWRT专版.html`

- 进入路由器的界面一般为192.168.1.1 点击 系统->软件包->过滤器->mwan3 下载`luci-i18n-mwan3-zh-cn`
  或者使用shell连接路由器输入
  ```shell
  opkg install mwan3
  opkg install luci-app-mwan3
  opkg install luci-i18n-mwan3-zh-cn
  ```
下载完成后界面会出现 网络->负载均衡

#### MWAN-接口配置详解
`！！！！添加的成员名称尽量与WAN口的名称一样`
- 跟踪的主机或IP 输入国内的如：114.114.114.114 233.5.5.5 119.29.29.29等等
- 其他的默认配置或适当修改就可以

#### MWAN-成员配置详解
`成员名称的明明规范为 XXX_m1_w1` XXX为上一步设置的接口名称
- 选择接口
- 设置跃点数和比重：跃点设置为1，比重根据自己的带宽设置（建议设置为1）

#### 策略
- 添加一个策列并将所有的成员添加加进去

#### 规则
- 使用默认的default-rule规则即可

###使用Shadowsocks+ChinaDNS实现代理
- 参考资料：`https://cokebar.info/archives/664#comments`

    `\Openwrt X86_64完整配置\参考资料\Shadowsocks+chianDNS 实现OpenWRT 路由器特殊IP代理.html`

- 添加所需的依赖包
```shell
libgcc # target/cpu型号/package目录下
libpthread # 同上
ip-full # base
ipset # base 无此包不能使用luci-app-shadowsocks
      #      只能使用luci-app-shadowsocks-without-ipset 性能会下降
iptables-mod-tproxy # base 无此包将无法代理UDP流量
zlib # base
```
- 检测是否全部安装
```shell
opkg install ip-full ipset iptables-mod-tproxy libpthread
```
- 查看CPU内核架构
```shell
opkg print-architecture
```

- 下载ss和ChinaDNS：`http://openwrt-dist.sourceforge.net/packages/`（安装包里有下载好的2018-08-03版本的）

下载列表：    
shadowsocks-libev_x.x.x-x_xxxx.ipk    
dns-forwarder_x.x.x-x_xxxx.ipk  
ChinaDNS_x.x.x-x_xxxx.ipk   
luci-app-shadowsocks_x.x.x-x_all.ipk  
luci-app-chinadns_x.x.x-x_all.ipk   
luci-app-dns-forwarder_x.x.x-x_all.ipk

- 将下载好的ipk文件用sftp传送到路由器中
```shell
opkg install /目录名/软件名
```
完成软件安装

#### 配置SS代理
- 服务->影梭    

服务器管理：添加要代理的服务器   

访问控制：外网区域    
    被忽略IP列表选择ChinaDNS路由表
    设置要强制代理的IP（强制代理不会受到ChinaDNS路由表的影响）

访问控制：内网区域 
    接口 只需要选择 桥接lan口

    1.正常代理：使用外网区域设置的方式。

    2.直接连接：忽略外网区域的设置，不走代理。

    3.全局代理：忽略外网区域的设置，强制走代理。

访问控制：基本设置

  `只需要设置透明代理即可 `
  
运行状态：

透明代理： ss-redir的运行状态，shadowosocks代理的基础服务，提供TCP/UDP透明代理。

socks5代理：ss-local的运行状态，shadowsocks的socks5代理功能，路由器作为socks5代理服务器。

端口转发：ss-tunnel的运行状态，通常用作转发DNS。

透明代理：

主服务器：透明代理使用的默认服务器（TCP透明代理、端口转发两个功能会使用这里选择的服务器），此处从列表中选择你需要使用的服务器。

UDP服务器：UDP透明代理使用的服务器，可以和主服务器一致，也可以不同（也就是可以使用不同的服务器分别代理TCP和UDP连接）。在代理一些外服网络游戏的时候可能比较有用，其他多数情况下可以关闭。开启此项需要服务器支持，服务器端需要开启UDP功能。

本地端口：shadowsocks服务需要占用一个路由器端口，推荐保留默认，但不能和其他运行的程序有冲突，也不能和下面”socks5代理”和“端口转发”中的本地端口冲突。

更新MTU：手工指定MTU值传递给shadowsocks。通常PPPoE宽带连接为1492（也是此处默认值），网线直连以太网为1500。通过执行 ifconfig 可查看MTU值（找到wan口对应网卡设备即可）。也可以通过ping命令测试，具体步骤请超出本文讨论范围，不再叙述。

socks5代理：

有需要就开吧，不知道有用没的话就停用。

端口转发：

通常用于转发DNS请求，本文的方法未用到此功能，不用开启。如需要UDP方式的国外DNS解析，可用这个功能替换掉dns-forwarder.

最后再次保存&应用，并刷新页面，看到“运行状态”中对应的项目显示运行中即表示成功。

### shadowosocks 支持UDP转发

- 首先Openwrt需要下载 iptables-mod-tproxy,kmod-ipt-tproxy和ip-full依赖包
- 服务器端也需要支持UDP转发     
    配置如下
    ```shell
    打开/etc/sysctl.conf这个文件，取消掉 net.ipv4.ip_forward=1 这一行的注释.
    然后执行
    sudo sysctl -p
    使修改后的文件配置立即生效。
    ```


#### chinaDNS
- 因为下载的chainaDNS所附带的忽略文件是比较老旧的版本因此需要手动更新
```shell
wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > /etc/chinadns_chnroute.txt
```
- 更新完成最好重启SS服务