---
文章标题: "[[04_补充：extboot分区解释说明]]" 
文章作者: Dakkk
文章概要: |
  分析U-Boot和内核阶段对extboot分区的不同访问目的，U-Boot只读加载内核启动，内核阶段可读写维护更新
tags:
- "U-Boot"
- "extboot分区"
- "内核启动"
- "设备树"
- "ext4文件系统"
- "嵌入式系统"
- "分区挂载"
- "系统启动"
相关文章:
- "[[06_U-boot的修改与编译]]"
- "[[1_驱动关键技术]]"
- "[[05_U-boot的介绍]]"
- "[[11_设备树的编译]]"
- "[[12_根文件系统的介绍]]"
文章分类: "🐧 Linux系统"
文章路径: "06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/2_镜像构建与部署/04_补充：extboot分区解释说明.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-06 18:34:14
修改时间: 2025-05-28 00:19:51
---


**U-Boot 阶段挂载的 extboot 分区与内核阶段的 extboot 分区是同一个物理分区**，但两者的**访问目的和操作内容不同**。以下是具体分析：

## 1 物理分区相同

- **分区定义**：  
    extboot 是系统镜像中的一个独立分区（通常为 `boot` 分区），格式为 **ext4**，存储以下内容：
    - 内核镜像（`Image` 或 `zImage`）
    - 多板卡设备树（`*.dtb`）
    - 设备树插件（`*.dtbo`）
    - 内核 deb 包（`linux-image-*.deb`）
    - 环境变量文件（`uEnv.txt`）

- **物理位置**：
    - 在存储介质（如 eMMC、SD 卡）中被分配为固定分区（例如 `/dev/mmcblk0p2`）
## 2 Uboot阶段挂载extboot分区的目的

- **核心任务**：
    - **加载内核和设备树**：  
        U-Boot 挂载 extboot 分区后，根据硬件 ID 选择对应的设备树文件（`*.dtb`），并加载内核镜像到内存。
    - **传递启动参数**：  
        通过 `uEnv.txt` 或环境变量设置内核启动参数（如 `root=PARTUUID=...`）。

- **操作特点**：
    - **只读访问**：U-Boot 仅需读取内核镜像和设备树文件，不修改分区内容。
    - **依赖 ext4 驱动**：U-Boot 需支持 ext4 文件系统驱动（配置 `CONFIG_FS_EXT4=y`）

**示例流程**
```shell
# U-Boot 命令行操作
ext4load mmc 0:2 ${kernel_addr_r} Image           # 从 extboot 加载内核
ext4load mmc 0:2 ${fdt_addr_r} rk3568-lubancat2.dtb # 加载设备树
bootm ${kernel_addr_r} - ${fdt_addr_r}              # 启动内核
```
## 3 内核阶段挂载 extboot 的目的

- **核心任务**：
    - **加载设备树插件**：运行时动态加载 `.dtbo` 文件扩展硬件配置。
    - **在线更新内核**：通过 `apt` 或 `dpkg` 安装新的 `linux-image-*.deb` 包，更新内核镜像和设备树。
    - **维护配置文件**：存储内核配置（`.config`）、`System.map` 等调试文件。

- **操作特点**：
    - **读写访问**：内核（或用户空间）可能需要修改 extboot 内容（如更新内核、添加插件）。
    - **挂载点**：通常挂载到 `/boot` 目录（用户可通过 `/boot` 路径直接操作文件）。

**示例流程**：
```shell
# 内核启动后，用户空间操作
sudo mount /dev/mmcblk0p2 /boot          # 挂载 extboot 到 /boot
sudo dpkg -i /boot/linux-image-4.19.xxx.deb  # 安装新内核
sudo cp custom.dtbo /boot/overlays/      # 添加设备树插件

```
## 4 两者的核心区别

| **维度**   | **U-Boot 阶段挂载 extboot** | **内核阶段挂载 extboot**         |
| -------- | ----------------------- | -------------------------- |
| **访问目的** | 加载内核和设备树，启动系统           | 维护内核、插件、在线更新               |
| **操作权限** | 只读                      | 读写（可修改内容）                  |
| **挂载点**  | 无固定挂载点（通过设备路径直接访问）      | 通常挂载到 `/boot`              |
| **依赖组件** | U-Boot 的 ext4 驱动        | 内核的 ext4 驱动                |
| **典型操作** | `ext4load` 命令加载文件       | `mount`、`apt`、`dpkg` 等工具操作 |
## 5 总结

通过这种设计，extboot 分区在系统全生命周期中承担了**启动与维护的双重角色**，兼顾灵活性和效率
![assets_task_01jtjjnd24ehyanbw5d06dk0gr_1746527813_src_0.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/cd088d4a392c0ce371a18f1de068b19c.png)



