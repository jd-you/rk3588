# IMX6ULL 移植最新uboot
https://blog.csdn.net/lgc1990/category_10487311.html

视频教程：https://www.bilibili.com/video/BV1Pt411n7cv/?spm_id_from=333.337.search-card.all.click&vd_source=be8f8a21c85cd61e0f830bde5567648b

## 交叉编译
https://blog.csdn.net/qq_36347513/article/details/126658866
### Linaro
https://releases.linaro.org/components/toolchain/binaries/
1. 下载  `wget https://releases.linaro.org/components/toolchain/binaries/6.4-2017.08/arm-linux-gnueabihf/gcc-linaro-6.4.1-2017.08-x86_64_arm-linux-gnueabihf.tar.xz`

介绍uboot的移植：https://zhuanlan.zhihu.com/p/110613703\
介绍imx（这篇很重要）：https://blog.csdn.net/zhaoyun_zzz/article/details/84990606\

重定位是什么？


Linux的启动地址是如何确定的？
1. iminfo：0x82000000 该值是默认值，可以再Kconfig中看到
2. armv7的_start在vectors.S中

为了查看emmc的分区，我们先用tftp启动linux：https://zhuanlan.zhihu.com/p/556094760
发现uboot以太网不通: https://blog.csdn.net/Wang_XB_3434/article/details/130817581?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-130817581-blog-128525475.235%5Ev38%5Epc_relevant_sort_base1&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-130817581-blog-128525475.235%5Ev38%5Epc_relevant_sort_base1&utm_relevant_index=2

eth：https://blog.csdn.net/qq_40684669/article/details/128586214

文件依賴：https://blog.csdn.net/JACK240541595/article/details/106635759

不錯的資料:https://github.com/zhaojh329/U-boot-1/blob/master/%E7%AC%AC2%E7%AB%A0-U-boot%E8%AE%BE%E5%A4%87%E6%A0%91.md
https://github.com/Staok/ARM-Linux-Study/blob/main/%E3%80%900%20ARM%20%26%20Linux%20%E4%B8%BB%E7%BA%BF%E5%89%A7%E6%83%85%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0%E3%80%91/%E4%B8%BB%E7%BA%BF%E5%89%A7%E6%83%8503-NXP-i.MX%E7%B3%BB%E5%88%97%E7%9A%84u-boot%E7%A7%BB%E6%A4%8D%E5%9F%BA%E7%A1%80%E8%AF%A6%E8%A7%A3/%E4%B8%BB%E7%BA%BF%E5%89%A7%E6%83%8503-NXP-i.MX%E7%B3%BB%E5%88%97%E7%9A%84u-boot%E7%A7%BB%E6%A4%8D%E5%9F%BA%E7%A1%80%E8%AF%A6%E8%A7%A3.md

沒有設置mac地址導致失敗：https://blog.csdn.net/qq_15269787/article/details/120346505

setenv ethaddr/eth1addr 00:01:3f:2d:3e:4d
setenv ethaddr 00:01:3f:2d:3e:4d
setenv ipaddr 192.168.2.2
setenv gatewayip 192.168.2.3
setenv netmask 255.255.255.0
setenv serverip 192.168.2.1
saveenv

# uboot 启动linux
在 U-Boot 中跳转到 Linux 内核的启动过程通常需要两个关键步骤：

1. **设置内核启动参数（kernel command line）：**
   在 U-Boot 中，你需要设置 Linux 内核启动时所需的命令行参数，例如根文件系统的位置、内核参数等。使用 `setenv` 命令在 U-Boot 中设置这些参数，例如：
   ```bash
   setenv bootargs root=/dev/sda1 console=ttyS0,115200
   ```
   这个命令会设置内核启动参数，具体的参数设置根据你的系统配置而定。

2. **加载内核镜像并启动：**
   在 U-Boot 中使用 `load` 命令加载 Linux 内核的镜像文件（通常是 `uImage` 或 `zImage`），然后使用 `bootm` 命令启动内核。例如：
   ```bash
   load mmc 0:1 ${loadaddr} uImage
   bootm ${loadaddr}
   ```
   这个命令会将从 MMC 卡（假设在 U-Boot 中识别为 `mmc 0:1`）加载的内核文件 `uImage` 放到内存地址 `${loadaddr}`，然后使用 `bootm` 启动内核。

确保 `${loadaddr}` 变量设置为足够大的内存地址，以容纳加载的内核镜像。这些命令的具体语法和参数可能会根据你的硬件平台和 U-Boot 版本而有所不同，所以请查阅你的硬件和 U-Boot 的文档以获取准确的命令和参数。

在你的系统中，U-Boot 的提示符可能是不同的，例如 `U-Boot>`。在输入命令时，请确保使用正确的提示符。

## uImage和zImage的区别
`zImage` 和 `uImage` 是 Linux 内核的两种不同的压缩格式，用于嵌入式系统中的引导加载过程。它们的主要区别在于压缩算法和支持的功能。

### 1. **zImage：**

`zImage` 是一种简单的内核压缩格式，使用的压缩算法是 gzip。它是 Linux 内核的基本压缩格式，可以被几乎所有的启动加载程序（包括 U-Boot）所支持。`zImage` 文件包含了压缩的内核映像以及一个解压缩的程序。在引导过程中，引导加载程序负责解压缩 `zImage` 并将内核加载到内存中，然后启动内核。这种格式简单、通用，适用于大多数嵌入式系统。

### 2. **uImage：**

`uImage` 是 U-Boot 引导加载程序特定的内核压缩格式。它支持多种压缩算法，包括 gzip、bzip2 和 lzma。`uImage` 文件不仅包含了压缩的内核映像，还包含了启动内核所需的元数据信息，例如内核的加载地址、命令行参数等。这种格式允许 U-Boot 在加载内核之前，自动解压缩内核，并使用正确的启动参数。这样可以大大简化引导过程，因为 U-Boot 负责处理所有必要的操作。

**总结：**

- `zImage` 是 Linux 内核的基本压缩格式，简单通用，适用于几乎所有的引导加载程序。
- `uImage` 是 U-Boot 引导加载程序特有的内核压缩格式，包含了压缩的内核映像和启动内核所需的元数据信息，使得引导过程更加简化和自动化。

在选择使用哪种格式时，你需要考虑你的嵌入式系统的需求以及引导加载程序的支持能力。如果你的系统使用 U-Boot 作为引导加载程序，那么使用 `uImage` 格式可能会更加方便。

## 如何加载zImage
在 U-Boot 中加载 `zImage` 内核映像通常涉及以下几个步骤：

1. **设置 U-Boot 环境变量：**
   在 U-Boot 中，你需要设置用于加载 `zImage` 的环境变量，包括 `loadaddr`（加载地址）和 `bootargs`（内核启动参数）。例如：
   ```bash
   setenv loadaddr 0x80008000  # 加载地址通常是内存中的一个合适位置
   setenv bootargs root=/dev/mmcblk0p2 console=ttyS0,115200  # 设置内核启动参数
   ```

2. **从存储介质加载 `zImage` 文件：**
   使用 U-Boot 的 `load` 命令从存储介质（例如 SD 卡、NFS、TFTP 服务器等）加载 `zImage` 文件到内存中，加载命令通常是这样的：
   ```bash
   load mmc 0:1 ${loadaddr} zImage
   ```
   这个命令会从 MMC 卡的第一个分区（`mmc 0:1`）加载 `zImage` 文件到内存地址 `${loadaddr}`。

3. **启动 Linux 内核：**
   一旦 `zImage` 文件被加载到内存中，使用 U-Boot 的 `bootz` 命令启动内核。例如：
   ```bash
   bootz ${loadaddr} - ${fdtaddr}
   ```
   这个命令会启动加载到 `${loadaddr}` 地址的 `zImage` 内核映像，并使用设备树（Device Tree）地址 `${fdtaddr}`（如果你的系统使用设备树的话）。

以上命令是基本的示例，具体的命令和地址可能会根据你的硬件平台、U-Boot 版本和存储介质而有所不同。请参考你的硬件和 U-Boot 的文档以获取准确的命令和地址。

记得在启动 `zImage` 内核之前，确保 `bootargs` 和其他环境变量被正确设置，以便 Linux 内核能够正确初始化系统。


## 嵌入式linux所包含的组件
kernel，设备树，rootfs和uboot\
uboot启动命令的解析：https://blog.csdn.net/xxxx123041/article/details/119938816 \
[Linux rootfs挂载过程](https://blog.csdn.net/zzZhangYiLong/article/details/125844927?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-125844927-blog-119938816.235^v38^pc_relevant_default_base3&spm=1001.2101.3001.4242.1&utm_relevant_index=3)

### 构建根文件系统
下载busybox
``` bash
 wget https://www.busybox.net/downloads/busybox-1.32.0.tar.bz2
 tar -jxvf ./busybox-1.32.0.tar.bz2
```
[使用busybox构建根文件系统以及过程中遇到的问题](https://blog.csdn.net/u013171226/article/details/129437616)\

[uboot下使用nfs网络下载kernel](https://blog.csdn.net/qq_42212668/article/details/125250873)
```
mkdir dev lib etc proc sys //创建必要的系统目录。
mkdir home tmp var mnt //创建可选的目录
```
### NFS
在u-boot下使用nfs时发现如下错误
```
=> nfs 0x82000000 /home/jiadiyou/Workspace/rootfs/test
Using ethernet@2188000 device
File transfer via NFS from server 192.168.2.4; our IP address is 192.168.2.2
Filename '/home/jiadiyou/Workspace/rootfs/test'.
Load address: 0x82000000
Loading: *** ERROR: File lookup fail
```
网上的解决方案：[Linux网络下载镜像时“nfs报错：ERROR: File lookup fail”解决方法](https://blog.csdn.net/qq_35333978/article/details/107288293)\
修改后仍然报错，查看`nfsstat`:
```
jiadiyou@jiadiyou-ThinkPad-T14-Gen-1:~/Workspace/my-uboot$ nfsstat
Server rpc stats:
calls      badcalls   badfmt     badauth    badclnt
40         0          0          0          0

Server nfs v3:
null             getattr          setattr          lookup           access
0         0%     0         0%     0         0%     40      100%     0         0%
readlink         read             write            create           mkdir
0         0%     0         0%     0         0%     0         0%     0         0%
symlink          mknod            remove           rmdir            rename
0         0%     0         0%     0         0%     0         0%     0         0%
link             readdir          readdirplus      fsstat           fsinfo
0         0%     0         0%     0         0%     0         0%     0         0%
pathconf         commit
0         0%     0         0%
```
可以看到只有v3在工作，而且`lookup`次数和我们操作的次数相同，基本上可以确定还是nfs协议版本的问题。查看u-boot代码，发现u-boot的nfs默认使用v3协议。作如下修改后重新编译u-boot即可解决问题。
```
--- a/net/nfs.c
+++ b/net/nfs.c
@@ -87,7 +87,7 @@ enum nfs_version {
        NFS_V3 = 3,
 };
 
-static enum nfs_version choosen_nfs_version = NFS_V3;
+static enum nfs_version choosen_nfs_version = NFS_V2;
```
nfs重要的命令：
```
sudo vim /etc/exports
sudo exportfs -a
sudo vim /etc/default/nfs-kernel-server
sudo systemctl restart nfs-kernel-server
nfsstat
```

### NFS挂载文件系统并启动linux
[IMX6ULL驱动开发前奏三：根文件系统构建步骤明细](https://blog.csdn.net/qq_43940175/article/details/124560883)

```
setenv bootargs 'console=ttymxc0,115200 root=/dev/nfs nfsroot=192.168.2.4:/home/jiadiyou/Workspace/rootfs/,proto=tcp rw ip=192.168.2.2:192.168.2.4:192.168.2.1:255.255.255.0::eth0:off'
```

```
tftp 0x82000000 zImage; tftp 0x83000000 100ask_imx6ull_mini.dtb; bootz 0x82000000 - 0x83000000
```
[制作最小根文件系统使用NFS挂载、烧写至EMMC两种方式](https://blog.csdn.net/weixin_43937576/article/details/109291816)

### NFS挂载文件系统，启动Linux后查看EMMC的分区内容
Linux启动后，`/dev`中没有emmc设备，无法查看上面的内容。怀疑是设备树的问题，编译原生SDK的设备树：
```
~/100ask_imx6ull_mini-sdk$ cd Linux-4.9.88
~/100ask_imx6ull_mini-sdk/Linux-4.9.88$ make mrproper
~/100ask_imx6ull_mini-sdk/Linux-4.9.88$ make 100ask_imx6ull_mini_defconfig
~/100ask_imx6ull_mini-sdk/Linux-4.9.88$ make zImage -jN
~/100ask_imx6ull_mini-sdk/Linux-4.9.88$ make dtbs
~/100ask_imx6ull_mini-sdk/Linux-4.9.88$ cd ~/100ask_imx6ull_mini-sdk/Linux-4.9.88/arch/arm/boot/dts
```
[ARM嵌入式——制作根文件系统并使用NFS挂载运行。](https://blog.51cto.com/u_6043682/3705038)\
[Linux和Uboot下eMMC boot分区读写](https://huaweicloud.csdn.net/635665dcd3efff3090b5d32b.html?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Eactivity-1-123995925-blog-80936361.235%5Ev38%5Epc_relevant_default_base3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Eactivity-1-123995925-blog-80936361.235%5Ev38%5Epc_relevant_default_base3&utm_relevant_index=2)\
[【uboot】MMC的代码分布](https://forums.100ask.net/t/topic/1217)\
[U-Boot命令之FAT 格式文件系统操作命令](https://blog.csdn.net/weixin_45309916/article/details/109185273)\
[U-Boot命令之EXT 格式文件系统操作命令](https://blog.csdn.net/weixin_45309916/article/details/109185736)\
[uboot命令使用学习（5）](https://blog.csdn.net/qq_41545736/article/details/125090606)
[IMX6ULL uboot命令](https://blog.csdn.net/weixin_45309916/category_10274949.html)
## Others
### 解压缩
解压tar.gz文件
```
tar -zxvf ×××.tar.gz
```
解压tar.bz2文件
```
tar -jxvf ×××.tar.bz2
```