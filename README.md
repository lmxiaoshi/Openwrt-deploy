# 刷Openwrt系统的注意是事项及过程
- 路由器去Openwrt官网找到自己路由器对应的版本好，固件升级就可以
- 下面主要介绍电脑刷X86_64的过程
## Openwrt X86_64

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
