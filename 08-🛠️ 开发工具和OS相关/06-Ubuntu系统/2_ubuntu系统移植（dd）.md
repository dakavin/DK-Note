---
文章标题: "[[2_ubuntu系统移植（dd）]]"
文章作者: Dakkk
文章概要: |
  文章图文并茂地介绍了Ubuntu系统移植至新硬盘的准备工作，包括使用DiskGenius进行分区和balenaEtcher制作Live USB启动盘。但文章关键的`dd`命令移植步骤缺失，导致教程不完整。
tags:
  - Ubuntu
  - 系统迁移
  - 硬盘分区
  - DiskGenius
  - balenaEtcher
  - Live USB
  - dd命令
相关文章:
  - "[[1_开发环境搭建]]"
  - "[[../../06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/4_Linux驱动开发实战/1_Linux驱动基础知识(重点)/1_驱动章节实验环境搭建]]"
  - "[[14_Ubuntu根文件系统构建（Skip）]]"
  - "[[2_Linux入门基础]]"
  - "[[2_Ubuntu的使用]]"
文章分类: 🛠️ 开发工具和OS相关
文章路径: 08-🛠️ 开发工具和OS相关/06-Ubuntu系统/2_ubuntu系统移植（dd）.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐ 基础知识
创建时间: 2025-05-22 16:41:31
修改时间: 2025-05-28 01:03:09
---

参考连接：
- [数据无忧：Ubuntu 系统迁移备份全指南_ubuntu系统备份-CSDN博客](https://blog.csdn.net/qq_39517117/article/details/140216351?ops_request_misc=%257B%2522request%255Fid%2522%253A%25227638fee3dd93cd90b32085ab8420bbd3%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=7638fee3dd93cd90b32085ab8420bbd3&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-140216351-null-null.142^v102^pc_search_result_base4&utm_term=Ubuntu%E7%B3%BB%E7%BB%9F%E8%BF%81%E7%A7%BB&spm=1018.2226.3001.4187)
- [Ubuntu 20.04系统搬家，迁移至更大容量硬盘_ubuntu系统迁移到新硬盘-CSDN博客](https://blog.csdn.net/wenquantongxin/article/details/130192728)
## 1 新盘分区

使用工具：DiskGenius

**1、清空硬盘**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/511068f290219d3fa8a0aa5e172f3f8d.png)

**2、创建引导分区**
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/35902b74c41d62a9988db578b87a7a84.png)

**3、创建其他分区**
在灰色界面右键，选择新建分区即可
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/807097474d3224a9f3f18f1d0c45dc4b.png)
- Linux Swap分区：和内存一样的大小即可 ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/6d1818a2f9ae6c0965fd1055f3024155.png)
- 创建根目录 `/` 分区 ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/e5b8feaecc4c43f60f9eb58b392dcaf7.png)
- 创建 `/home` 目录分区 ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/78d699f102871f7f9101c7e94e512c64.png)
- 剩下空间创建 `/var` 目录分区 
- 注意要保存更改
- 最后结果如下图 ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/8d66aa5d7ad4bae28d3f1861f4820189.png)
## 2 制作启动盘

ubuntu官网下载镜像：
- [Ubuntu系统下载 | Ubuntu](https://cn.ubuntu.com/download)

etcher官网下载U盘制作工具：
- [balenaEtcher - Flash OS images to SD cards & USB drives](https://etcher.balena.io/)

准备一个U盘，按照软件的提示制作U盘启动工具
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/31ecb87bee7a15abaa91c52986a173ed.png)


开机按键进入bios模式，选择U盘启动即可进入Live模式

## 3 开始移植

移植命令如下
```shell

```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/4d224a4881711e7e34f120941709cf5e.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/d303e249f40497bb3358e35b9e35b156.png)

