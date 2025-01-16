---
title: 华人运通USB升级与NFS升级操作
date: 2024-07-09 12:07:09
tags: 工作
---

## 将编译好的android系统包制作USB升级包，NFS升级包。

**1.环境准备：**

* 以下全部为Linux操作

**注意：车机只支持U盘格式为EXT4格式**

* 安装需要的工具

  ```shell
  sudo apt-get install lftp
  sudo apt-get install simg2img
  ```

* 下载一个weakly版本，并解压，之后解压当前目录下b_Package_USB和c_Package_NFS，全部完成后目录如下：

  ![image-20211230142902612](/typora-user-images/image-20211230142902612.png)

* 下载一个编译完成的包，并解压，目录如下：

  ![image-20211230143543554](/typora-user-images/image-20211230143543554.png)

***

**2.制作USB升级包**

* 进入编译完成的包所在路径打开终端

* 复制vendor.img为la_vendor.img

  ```shell
  cp vendor.img la_vendor.img
  ```

* 使用simg2simg对system.img和vendor.img进行转换

  ```shell
  simg2img system.img la_system.img
  simg2img vendor.img xx_vendor.img
  ```

* 完成后文件清单应为:

  ```shell
  cache.img
  log.img
  persist.img
  private.img
  system.img
  userdata.img
  vendor.img
  la_vendor.img
  la_system.img
  xx_vendor.img
  ```

* 进入weakly版本路径01.02.5062.X10C > HOST > android

* 上面的完成清单，其中四个替换weakly版本路径下>b_Package_USB/UpdatePackage/ARM中的文件:

  ```shell
  la_system.img
  la_vendor.img
  log.img
  persist.img
  ```

* 到此U盘升级包UpdatePackage，就已经制作完毕，U盘下只需要放UpdatePackage文件即可

***

**3.接下来需要将UpdatePackage压缩制作NFS升级包**

* 在b_Package_USB路径下打开终端,执行命令:

  ```shell
  tar -cvzf UpdatePackage.tar.gz UpdatePackage
  ```

* 将UpdatePackage.tar.gz移动到c_Package_NFS中

* 在c_Package_NFS中执行以下命令，复制生成的哈希码粘贴到nfs_hash.txt

  ```shell
  sha256sum UpdatePackage.tar.gz 
  ```

* 如果使用NFS升级的方式，U盘下应有“nfs_hash.txt”文件和“UpdatePackage.tar.gz”文件即可

**注意：压缩包格式必须为“tar.gz”，否则无法成功升级。**

***

**4.使用USB升级的方式，需要U盘连接副屏，进入工程模式升级**

副屏进入工程模式以及后续操作如下：

（1）程序>

（2）设置>

（3）关于>

（4）点击三下“系统信息”>

（5）点击一下“容量信息”>

（6）点击一下“系统信息”>

（7）点击一下“容量信息”>

（8）长按“容量信息”下方“图片”>

（9）多次点击“车牌号码”>

（10）长按“USB UPGRADE”>点击"确定"

***

**5.使用NFS升级的方式，需要U盘连接主屏，主屏连接副屏，进入工程模式升级**

副屏操作与上述USB升级方式一致。上述步骤10不同，操作为：

点击“NFS UPGRADE”

6.制作OTA包

复制la_system和xx_vendor

xx_vebdor------>VX1_FSE_ARM_la_vendor_01025049X10C.img

la_system------->VX1_FSE_ARM_la_system_01025049X10C.img

