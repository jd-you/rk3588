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