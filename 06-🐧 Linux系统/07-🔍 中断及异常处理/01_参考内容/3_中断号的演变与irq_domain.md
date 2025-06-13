---
文章标题: "[[3_中断号的演变与irq_domain]]"
文章作者: Dakkk
文章概要: |
  文章阐述Linux内核中断处理从固定中断号到动态分配的演进，核心引入irq_domain。它解耦了中断号与硬件，支持层次化中断控制器并整合设备树，显著提升系统灵活性、可维护性与扩展性。
tags:
  - Linux内核
  - 中断处理
  - irq_domain
  - 设备树
  - 中断控制器
  - 硬件抽象
  - 驱动开发
相关文章:
  - "[[../../09-🏗️ Linux设备模型与设备树(重点)/3_设备树基础/05_设备树中时钟描述 (被使用)]]"
  - "[[../../09-🏗️ Linux设备模型与设备树(重点)/3_设备树基础/1_参考内容/中断系统中的设备树]]"
  - "[[1_驱动关键技术]]"
  - "[[../../09-🏗️ Linux设备模型与设备树(重点)/3_设备树基础/03_图解Kernel Device Tree(设备树)的使用]]"
  - "[[../../09-🏗️ Linux设备模型与设备树(重点)/3_设备树基础/04_设备树中CPU描述 (不需要改)]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/01-⚡ 中断及异常/01_基于韦神设备树课程/3_中断号的演变与irq_domain.md
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2025-06-10 20:34:00
修改时间: 2025-06-10 21:46:12
---

## 1 中断号演变的背景(传统中断处理方式)

以前中断号(virq)跟硬件密切相关，现在的趋势是中断号跟硬件无关，仅仅是一个标号而已

以前，对于每一个硬件中断(hwirq)都预先确定它的中断号(virq)，这些中断号一般都写在一个头文件里，比如 `内核源码/arch/arm/mach-s3c24xx/include/mach/irqs.h`
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f419bcc731384252d2bdb035d36c9c1a.png)

**---------------使用流程---------------**
1. **请求中断时**
	```c
	request_irq(virq, my_handler)
	```
	- virq：虚拟中断号，就是上图中某一个宏（如IRQ_LCD等）
	- my_handler：自定义的中断处理函数
	- 内核根据virq可以知道对应的硬件中断，然后去设置、使能中断等
2. **发生硬件中断时**
	- 内核读取硬件信息，确定hwirq
	- 根据宏反算出virq
	- 然后调用`irq_desc[virq].handle_irq`，最终用到自己写的中断函数`my_handler`

从下图中可以看到，传统方式下每个硬件中断都有固定的中断号映射关系
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/144b956cf3ca90af33128d79fa9da558.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3c0bb8d32e28409444901e5079462962.png)

## 2 hwirq到virq的转换问题

### 2.1 核心问题

**怎么根据hwirq计算出virq？**
- 硬件上有多个intc(中断控制器)，对于同一个hwirq数值，会对应不同的virq
	- 如：上图中的第一层中断（virq = hwirq +1），第二次子中断器（virq = hwirq' + 36）
- 所以在将hwirq时，应该强调 `是哪一个intc的hwirq`

> [!tip]+ 提示
> intc表示某个中断控制器

### 2.2 irq_domain概念的引入

在描述hwirq转换为virq时，引入一个概念：`irq_domain(域)`，在这个域里hwirq转换为某个virq
- 不同的域有不同的转换方法
- 显然，irq_domain和intc是一一对应的
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/352930c9f646b54518b002883d8c6750.png)
## 3 传统方式的缺陷和改进

**---------------缺陷---------------**
当中断控制器越来越多、中断也越来越多时，传统方法（virq和hwirq固定绑定）有明显缺陷：
1. **工作量大**
	- 需要给每个中断确定它的中断号
	- 写出对应的宏定义，可能有成百上千个
2. **容易错误**
	- 要确保每个硬件中断对应的中断号互不重复


**---------------改进---------------**
1. **解耦合**：hwirq跟virq之间不在固定绑定
2. **动态分配**：要使用某个hwirq时
	- 先在irq_desc数组中找到一个空闲项，它的位置就是virq
	- 再在`irq_desc[virq]`中放置处理函数

## 4 新中断体系的使用方法

**---------------完整流程---------------**
1. **设备树声明**：以前是request_irq发起，现在是先在设备树文件中声明想使用哪一个中断（哪一个中断控制器下的哪一个中断）
2. **内核解析设备树**：
	- 根据“中断控制器”确定irq_domain
	- 根据“哪一个中断”确定hwirq
	- 在irq_desc数组中找出一个空闲项，它的位置就是vriq
	- 把virq和hwirq的关系保存在irq_domain中：`irq_domain.linear_revmap[hwirq] = virq`
	  ![20250608_115035_297_copy.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/b4ab6add439e50f0d6bb9a08152df497.png)
3. **驱动程序请求中断**：`request_irq(virq, my_handler)`
4. **中断发生时的处理**
	- 内核读取硬件信息，确定hwirq
	- 确定 `virq = irq_domain.linear_revmap[hwirq]`
	- 调用 `irq_desc[virq].handle_irq`，最终会用到my_handler

**---------------irq_desc空闲项查找办法---------------**
- 内核中定义了一个位图（数组）`allocated_irqs`，里面的每一位都对应`irq_desc`的每一项
	```txt
	allocated_irqs      --->          irq_desc
	    bit0            --->          irq_desc[0]
		bit1            --->          irq_desc[1]
		bit2            --->          irq_desc[2]
		bit3            --->          irq_desc[3]
		......
	```
	- 但bit0 = 1，表示被占用了
- 内核采用了一个取巧的办法，不会每次从0开始查找，会从hwirq开始查找

## 5 子中断控制器的处理

假设要使用**子中断控制器(subintc)的n号中断**，它发生时会导致**父中断控制器(intc)的m号中断**

**---------------设置阶段---------------**
1. **设备树配置**
	- 设备树表明要使用 `<subintc n>`
	- subintc表示要使用`<inic m>`
2. **内核解析设备树**：找到对应的virq
	- 为`<inic m>`找到空闲项`irq_desc[virq]`
		- `irq_domain.linear_revmap[m] = virq`
	- 设置`irq_desc[virq]`的handle_irq为某个分析函数demux_func
	- 为`<subintc n>`找到空闲项`irq_desc[virq']`
		- `sub irq_domain.linear_revmap[n] = virq'`

**---------------运行阶段---------------**
3. **驱动程序**：`request_irq(virq', my_handler)`
	- 这样会将my_handler保存在`irq_desc[virq'].action`链表中
4. **中断发生时**
	- 内核读取intc硬件信息，确定hwirq = m
	- 确定 `virq = irq_domain.linear_revmap[m]`
	- 调用`irq_desc[virq].handle_irq`，即demux_func
5. **demux_func执行**
	- 读取sub intc硬件信息，确定hwirq = n
	- 通过sub intc对应的irq_domain来确定 `virq' = irq_domain.linear_revmap[n]`
	- 调用 `irq_desc[virq'].handle_irq`，即my_handler

## 6 如何兼容以前hwirq和virq的绑定关系呢？

irq_domain通过**Legacy映射**来兼容传统的固定绑定关系：

1. **Legacy映射方式**：使用固定偏移量计算，`virq = hwirq + irq_base_offset`，保持传统的1:1映射关系
2. **多映射并存**：系统同时支持Legacy映射（兼容老代码）和Linear映射（新的动态分配）
3. **透明兼容**：老代码中的固定IRQ号定义（如`#define IRQ_TIMER0 32`）无需修改即可继续工作
4. **渐进升级**：允许系统逐步从传统方式迁移到新的irq_domain机制，不强制一次性切换

## 7 总结

新的中断体系通过引入irq_domain概念，实现了：
1. **硬件无关性**：中断号不再与硬件强绑定
2. **动态分配**：按需分配中断号，避免冲突
3. **层次化管理**：支持复杂的中断控制器层次结构
4. **灵活性**：适应各种复杂的硬件架构

这种设计大大简化了中断管理，提高了系统的可维护性和扩展性。