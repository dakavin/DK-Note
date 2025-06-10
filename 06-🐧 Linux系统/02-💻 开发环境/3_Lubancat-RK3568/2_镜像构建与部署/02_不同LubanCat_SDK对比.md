---
文章标题: "[[02_不同LubanCat_SDK对比]]" 
文章作者: Dakkk
文章概要: |
  介绍野火基于LubanCat-RK系列板卡推出的三个SDK版本，对比LubanCat_Gen_SDK和LubanCat_Chip_SDK的功能差异、更新状态和板卡支持情况。
tags:
- "LubanCat"
- "SDK"
- "RK3568"
- "嵌入式Linux"
- "野火开发板"
- "镜像构建"
- "固件开发"
- "硬件支持"
相关文章:
- "[[03_LubanCat_Gen_SDK]]"
- "[[09_修改启动logo（Skip）]]"
- "[[15_使用Docker构建根文件系统（Skip）]]"
- "[[05_U-boot的介绍]]"
- "[[14_Ubuntu根文件系统构建（Skip）]]"
文章分类: "🐧 Linux系统"
文章路径: "06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/2_镜像构建与部署/02_不同LubanCat_SDK对比.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-05-04 19:11:18
修改时间: 2025-05-28 00:19:51
---


> LubanCat_Chip_SDK已于2024.9.10停止更新，只做BUG修复。RK系列板卡后续开发支持转移到LubanCat_Linux_Generic_SDK。 各版本SDK独有内容将在标题添加 **(Gen)** 或 **(Chip)** 标识，未添加此标识则表示内容通用。

我们这里只考虑RK3568板卡，其他的不做描述，相关内容可以参考下面的链接
- [2. 不同LubanCat_SDK对比 — [野火]嵌入式Linux镜像构建与部署——基于LubanCat-RK系列板卡 文档](https://doc.embedfire.com/linux/rk356x/build_and_deploy/zh/latest/building_image/lubancat_sdk/lubancat_sdk_compare.html)
## 1 不同版本SDK对比

为了满足用户的需求，目前野火基于LubanCat-RK系列板卡共推出3个SDK包，分别是LubanCat_Linux_rk356x_SDK、LubanCat_Linux_rk3588_SDK和LubanCat_Linux_Generic_SDK。

### 1.1 区分SDK版本

本文档同时包含了多个SDK的使用说明，除了通用内容以外，还包含根据SDK差异编写的文档，为了更好的将本文档内容与不同的SDK包对应，我们先要区分SDK版本。

通过网盘下载SDK源码的用户，可以通过源码压缩包的名称来区分，分别是：
- LubanCat_Linux_Generic_SDK
- LubanCat_Linux_rk356x_SDK
- LubanCat_Linux_rk3588_SDK
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/08b7282f341634e519875d19a7e15c05.png)
通过Github在线下载或已经将SDK源码包解压的用户，可以通过repo工具的配置文件来查看。

查看.repo/manifest.xml文件中的include这一行，name后面的字段是实际的xml配置文件的名称
- lubancat_linux_generic.xml，对应LubanCat_Linux_Generic_SDK
- rk356x_linux_release.xml，对应LubanCat_Linux_rk356x_SDK
- rk3588_linux_release.xml，对应LubanCat_Linux_rk3588_SDK

### 1.2 SDK对比

在本文档中，将LubanCat_Linux_Generic_SDK简称为`LubanCat_Gen_SDK`； 由于LubanCat_Linux_rk356x_SDK和LubanCat_Linux_rk3588_SDK基于同一框架，差异较小，所以统称为`LubanCat_Chip_SDK`。 如果文档中的内容通用于所有版本SDK，则以`LubanCat-SDK`代指

|   |   |   |
|---|---|---|
|差异|LubanCat_Gen_SDK|LubanCat_Chip_SDK|
|支持板卡|所有使用RK芯片的LubanCat板卡|对应RK芯片的LubanCat板卡|
|更新状态|持续更新中|已进入稳定版本,只修复BUG|
|便利性|框架较新,功能更多,开发更便利|配置简单，冗余功能少|
|功能|使用较新的内核、根文件系统等组件|使用稳定版的内核、根文件系统等组件|
## 2 Gen_SDK板卡支持情况

**RK3568**
- 鲁班猫2系列,使用RK3568/RK3568J(工业级)主芯片
## 3 Chip_SDK板卡支持情况

**LubanCat_Linux_rk356x_SDK**
- 鲁班猫2系列,使用RK3568/RK3568J(工业级)主芯片

**注意：** 此SDK不支持使用rk3562/RK3562J主芯片的 LubanCat-1HS 板卡