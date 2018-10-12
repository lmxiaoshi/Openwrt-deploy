# 关于IPXE

- 官网：http://ipxe.org/
- github:git://git.ipxe.org/ipxe.git  

## 编译ipxe


- 准备工作： 
    1. 准备一个CentOS7 64位系统
    2. 安装官方要求的依赖。
    ```shell
        yum install -y gcc binutils make perl liblzma mtools mkisofs syslinux
    ```
    3. 克隆官网的ipxe仓库到`/usr/local/src`  
        ```shell
        cd /usr/local/src
        git clone git://git.ipxe.org/ipxe.git
        ```

-  编译过程：  

    1. 创建一个空文件，命名为`make_for_ipxe`，然后保存到`/tmp/make_for_ipxe`，在该文件中写入如下内容：
    ```shell
    #!ipxe
    dhcp
    chain http://${net0/gateway}/ipxe/boot.config  //该命令为将 net0的网关设置为IPXE的服务器地址  
    或者 chain http://192.168.x.x/ipxe/boot.config `必须为路由器的IP地址`
    ```
  - 我也放了我做好的文件在文件中

    2. 开始编译
        ```shell
        cd  /usr/local/src/ipxe/src
        make bin/undionly.kpxe EMBED=/tmp/make_for_ipxe.ipxe
        ```
	！！！！可能出现错误
	centos-5.8-x64 build ipxe failed
	util/zbin.c:7:18: error: lzma.h: No such file or directory
	解决：yum install xz-devel.x86_64

    3. 编译完成后把`undionly.kpxe` 复制出来
    ```shell
    cp /usr/local/src/ipxe/src/bin/undionly.kpxe /tmp/ # 把undionly.kpxe先复制到tmp目录中
    ```
    通过FTP把`/tmp/undionly.kpxe`文件，复制出来，放置到路由器中的`/ipxe/`目录下。

- 启动windwosPE系统还需要 wimboot文件可以在IPXE官网下载，放在web服务器中

## 配置ipxe启动项文件boot.config  

### 无菜单通过wimboot直接启动PE  
- 启动菜单分别配置了引导wim镜像的文件和iso镜像的文件
在`boot.config`文件中写入以下内容

```shell
kernel wimboot gui
initrd boot/fonts/chs_boot.ttf  chs_boot.ttf
initrd boot/fonts/wgl4_boot.ttf     wgl4_boot.ttf
initrd boot/bcd         BCD
initrd boot/boot.sdi    boot.sdi
initrd sources/boot.wim boot.wim
boot
```

### 有菜单启动
在`boot.config`文件中写入以下内容：

```shell
#!ipxe
  #set menu-timeout 8000
   set menu-default 0pe
   isset ${ip} || dhcp
   isset ${next-server} || set next-server 192.168.1.1

  menu iPXE Boot Menu
  item --gap --             --------------------------------- PE --------------------------------
  item HXT-IP192.168.1.154        HXT-IP 192.168.1.154
  item HaoXiTongPE-ISO            HaoXiTongPE-ISO
  item HaoXiTongPE                HaoXiTongPE
  item TinyCore                   TinyCore
  item 
  item --gap --             ---------------------------- Advanced options -----------------------
  item reboot               Reboot computer
  item --key x exit         Exit iPXE and continue BIOS boot                     -- x

  choose --timeout ${menu-timeout} --default ${menu-default} selected
  goto ${selected}


:reboot
  reboot

:exit
  exit


:HXT-IP192.168.1.154
initrd http://192.168.1.154/iso/mydisk.iso 
chain  memdisk iso raw || goto failed

:HaoXiTongPE-ISO
initrd iso/haoxitongpe.iso 
chain  memdisk iso raw || goto failed

:TinyCore
initrd iso/TinyCore.iso 
chain  memdisk iso raw || goto failed


:HaoXiTongPE
#kernel wimboot
kernel wimboot gui
initrd boot/fonts/chs_boot.ttf  chs_boot.ttf
initrd boot/fonts/wgl4_boot.ttf     wgl4_boot.ttf
initrd boot/bcd         BCD
initrd boot/boot.sdi    boot.sdi
initrd sources/boot.wim boot.wim
boot

```


    




## Shell 连接OpenWRT路由器配置

### 把路由器刷机成OpenWRT系统

刷机参考官方文档
- [设备固件列表](https://openwrt.org/toh/views/toh_fwdownload)  


- [刷家教程](https://wiki.openwrt.org/doc/howto/generic.flashing)

- [WNDR3800教程1](https://wiki.openwrt.org/toh/netgear/wndr3800)

- [WNDR3800教程2](https://wiki.openwrt.org/toh/hwdata/netgear/netgear_wndr3800 )  

-[USB挂载和U盘启动](http://www.cnblogs.com/double-win/p/3841801.html)

### 升级OpenWRT

1. 通过shell连接到OpenWRT
2. 运行下面的命令进行升级，该过程比较慢，一定耐心等待。升级失败的话，后面的命令是无法使用的。  **一定要看清楚，升级结果不能有一个错误**
    ```shell
    opkg update 
    ```
3. 安装中文语言包
    ```shell
    opkg install luci-i18n-base-zh-cn 
    ```    
4. 安装支持SFTP传输的服务
    ```shell
    opkg install vsftpd openssh-sftp-server # 支持sftp传输
    ```

5. 添加USB支持

    ```shell
    opkg install kmod-usb-core  #可选
    opkg install kmod-usb-uhci
    opkg install kmod-usb-storage
    opkg install kmod-usb2
    opkg install kmod-usb-ohci
    ```
6. 添加usb挂载，热插拔，以及boot支持
    ```shell
    opkg install block-mount          #挂载功能，在webui中的系统中会显示一个【挂载点】的菜单
    opkg install block-hotplug        #热插拔     
    opkg install block-extroot        #boot支持
    ```
7. 添加格式化和分区工具    
    ```shell     
    opkg install fdisk               #添加分区工具
    opkg install e2fsprogs          # 添加格式化和检测工具，支持mkfs    
    opkg install kmod-fs-ext4    #添加ext4文件系统支持 
    opkg install kmod-fs-ext3    #添加ext3文件系统支持    
    ```
8. 创建U盘分区，格式化U盘
搜索`fdisk`教程，来格式化U盘。把整个U盘格式化为`ext4`格式。 

格式化命令：
mkfs.ext4 /dev/sda1（硬盘的分区名称）

>通过以上配置已经实现了自动挂载功能，只需要在web网页中的`系统->挂载点`中添加一个挂载即可。挂载设备选择U盘，挂载路径填写`/www/ipxe/`。如果有问题，继续查看第9步，否则跳过第9步。  



9. 自动把U盘挂载到`/www/ipxe`目录下。  



配置 `/etc/config/fstab`，以支持开机自动挂载

```shell
block detect > /etc/config/fstab
/etc/init.d/fstab enable
```

获取U盘的`UUID`  

```shell
    block info
```

使用`vi`命令编辑`/etc/config/fstab`配置自动挂载。参考下面的配置示例：

```shell
config 'global'
        option  anon_swap       '0'
        option  anon_mount      '0'
        option  auto_swap       '1'
        option  auto_mount      '1'
        option  delay_root      '5'
        option  check_fs        '0'

config 'mount'
        option  target  '/www/ipxe'
        option  uuid    'ed5f921e-c4ef-4884-8ce7-4eeb0ef6f008'
        option  enabled '1'
```        

注意：需要把`option enabled` 从`0`改为`1` ，然后重启`fstab`服务。  

```shell
/sbin/block mount
```

6. 确认信息，以上配置完成后运行`df -h`应该看到以下信息：
```shell
root@root:~# df -h
Filesystem                Size      Used Available Use% Mounted on
/dev/root                 8.0M      8.0M         0 100% /rom
tmpfs                    61.1M    972.0K     60.1M   2% /tmp
/dev/mtdblock5            6.7M      1.3M      5.4M  20% /overlay
overlayfs:/overlay        6.7M      1.3M      5.4M  20% /
tmpfs                   512.0K         0    512.0K   0% /dev
/dev/sda1                14.2G     40.0M     13.4G   0% /www/ipxe
```
                     
### 传输文件

通过`Xftp`把`ipxe`目录下的所有内容上传到`/www/ipxe`目录下。


### TFTP路径基本配置
- TFTP 服务器根目录设置为 `/www/ipxe`  
- 网络启动镜像 设置为 `undionly.kpxe`

### 通过ISO启动注意事项

ISO需要可以引导ISO的引导程序进行启动，因此需要在之前步骤下载的syslinux中找去可以引导ISO的引导文件（memdisk）
在：
```shell
/usr/share/syslinux
```
将medisk通过sftp复制到本地然后上传到挂载目录中去

配置boot.config 启动命令

```shell
:HaoXiTongPE-ISO
initrd iso/haoxitongpe.iso 
chain  memdisk iso raw || goto failed
```