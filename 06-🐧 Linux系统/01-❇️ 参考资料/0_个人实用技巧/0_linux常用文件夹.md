---
文章标题: "[[0_linux常用文件夹]]"
文章作者: Dakkk
文章概要: |
  介绍Linux嵌入式开发中的关键目录结构，包括SDK源码路径、文件系统模块管理、设备节点路径，以及系统信息查看和设备管理的基础命令操作。
tags:
  - Linux文件系统
  - 嵌入式开发
  - 设备驱动
  - 内核模块
  - SDK目录结构
  - 系统信息查看
  - proc目录
  - sys目录
相关文章:
  - "[[08_Linux内核的编译]]"
  - "[[1_为什么学习Linux开发📕]]"
  - "[[3_嵌入式驱动开发学习路线(一口君)]]"
  - "[[../../08-📊 字符设备驱动开发/1_字符设备驱动模型基础(Lubancat)/6_Linux的设备驱动模型]]"
  - "[[_常用函数]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/01-❇️ 参考资料/0_个人实用技巧/0_linux常用文件夹.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-06-10 20:33:59
修改时间: 2025-06-13 12:48:47
---


**SDK相关地址**

内核地址：`~/LubanCat_SDK/kernel/`

设备树地址：`~/LubanCat_SDK/kernel/arch/arm64/boot/dts/rockchip`

驱动代码地址：`~/LubanCat_SDK/kernel/drivers`

pinctrl地址：`~/LubanCat_SDK/kernel/include/dt-bindings/pinctrl`

gpio宏定义地址：`~/LubanCat_SDK/kernel/include/dt-bindings/gpio`

**文件系统相关地址**

内核模块地址：`/lib/modules/内核版本/`
- 其中内核模块之间依赖的文件是 modules.dep
- 将自行创建的ko文件放在这个文件夹，然后使用 sudo depmod -a 即可将新的ko放入依赖文件中
- 要使其生效，还需要修改 `/etc/modules` 或者 `/etc/modules-load.d/<filename>.conf` 文件，将ko文件名（可以带ko或不带后缀）放入其中

以模块参数为名的文件地址：`/sys/module/模块名/parameters/` 

**字符设备**
- 字符设备节点：`/dev`
- 字符设备名称：`/proc/devices`

## 1 查看板卡系统信息

```shell
# 查看系统运行的信息（任务管理器）
ls /proc

# 在/proc文件夹下，有很多以数字命名的文件夹，这些文件夹是用来记录当前正在运行的进程状态， 文件名则是他们的pid号，每一个进程都对应一个pid号，用于辨识
# 查看任务进程
ls /proc/1

top

```

```shell
# 查看CPU信息
cat /proc/cpuinfo
# 查看内核版本
cat /proc/version
# 查看内存信息
cat /proc/meminfo
# 查看系统的内存大小
free
# 查看存储器容量
cat /proc/partitions
# 查看支持的文件系统
cat /proc/filesystems
# 查看CPU当前主频
cat /sys/devices/system/cpu/cpu0/cpufreq/cpuinfo_cur_freq
ls -g /sys/devices/system/cpu/cpu0/cpufreq/
```

## 2 初探 /sys 目录

/sys目录下的文件/文件夹向用户提供了一些关于设备、内核模块、文件系统以及其他内核组件的信息， 如：
- 子目录block中存放了所有的块设备
- 子目录bus中存放了系统中所有的总线类型，有i2c、usb、sdio、pci等
- 子目录class按类型归类设备，如leds、lcd、mtd、pwm等

尝试在板卡的终端执行以下命令查看sys各层级的目录内容：
```shell
#在板卡上的终端执行以下命令查看
ls /sys
ls /sys/class
ls /sys/class/leds
ls /sys/class/leds/sys_status_led
```

## 3 初探 /dev 目录

/dev目录也包含了非常丰富的设备信息，该目录下包含了Linux系统中使用的所有外部设备， 如`/dev/tty为串口设备`、`/dev/ram为内存`、通过这些设备文件，我们也可以访问到对应的硬件设备

尝试使用以下命令查看dev目录的内容：
```shell
ls /dev
ls /dev/input
```