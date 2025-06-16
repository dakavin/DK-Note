---
文章标题: "[[2_Linux对中断处理的框架及代码流程简述]]"
文章作者: Dakkk
文章概要: |
  文章详细阐述了Linux内核中断处理框架，从ARM异常向量表起始，经汇编到C语言转换，深入解析`irq_desc`核心结构，并结合共享、级联中断实例，揭示硬件中断如何层层分发至上层驱动，展现了内核分层与可扩展性设计。
tags:
  - Linux kernel
  - 中断处理
  - ARM架构
  - 异常向量表
  - irq_desc
  - 共享中断
  - 级联中断
  - 内核机制
相关文章:
  - "[[9_串行口通讯（重点）]]"
  - "[[../../07-🏗️ Linux设备模型与设备树(重点)/3_设备树基础/1_参考内容/内核对设备树的处理]]"
  - "[[3_嵌入式驱动开发学习路线(一口君)]]"
  - "[[3_中断号的演变与irq_domain]]"
文章分类: 🐧 Linux系统
文章路径: 06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/01-⚡ 中断及异常/01_基于韦神设备树课程/2_Linux对中断处理的框架及代码流程简述.md
文章难度: 高级 🔥
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2025-06-10 20:34:00
修改时间: 2025-06-10 21:46:18
---

## 1 学习目标

通过本章学习，我们将了解：
- Linux内核如何处理中断的完整流程
- 从硬件异常（中断）到C语言处理函数的转换过程
- 内核中断管理的数据结构设计
- 普通中断和级联中断的处理机制

## 2 中断处理的起点：异常向量表

### 2.1 硬件中断发生时的第一步

前面讲过，当中断发生时，CPU首先会跳转到异常向量表，这是中断处理的起点。

**文件位置**：`内核源码/arch/arm/kernel/entry-armv.S`
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c904da3eaef7808737b0203cac175cf5.png)

```s
/* arch\arm\kernel\entry-armv.S */
.section .vectors, "ax", %progbits
.L__vectors_start:
    W(b)    vector_rst      /* 复位异常 */
    W(b)    vector_und      /* 未定义指令异常 */
    W(ldr)  pc, .L__vectors_start + 0x1000
    W(b)    vector_pabt     /* 预取指令中止异常 */
    W(b)    vector_dabt     /* 数据访问中止异常 */
    W(b)    vector_addrexcptn
    W(b)    vector_irq      /* IRQ中断入口 */
    W(b)    vector_fiq      /* FIQ中断入口 */
```
- 这是ARM处理器的异常向量表
- 当发生IRQ中断时，CPU会自动跳转到`vector_irq`标签处开始执行
- `vector_irq`实际上是通过宏`vector_stub irq`来定义的
  
### 2.2 中断分发机制

`vector_irq`实际上是一个中断分发器，根据CPU被中断时的状态来决定调用哪个处理函数
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7cd541f44cb93fd147be9a417ebdfaf4.png)

**左侧-中断分发器（Interrupt dispatcher）**
```s
/*
* Interrupt dispatcher - 中断分发器
*/
vector_stub irq, IRQ_MODE, 4   
/* 相当于 vector_irq: ..., 
* 它会根据SPSR寄存器的值判断被中断时CPU的状态
* 然后调用对应的处理函数
*/
.long   __irq_usr               @  0  (USR_26 / USR_32)
.long   __irq_invalid           @  1  (FIQ_26 / FIQ_32)
.long   __irq_invalid           @  2  (IRQ_26 / IRQ_32)
.long   __irq_svc               @  3  (SVC_26 / SVC_32)
.long   __irq_invalid           @  4-f (其他无效模式)
...
```
- `vector_stub` 是中断处理的入口点，包含了一些列的`.long`指令，定义了不同中断号对应的处理函数，大部分指向无效中断处理`__irq_invalid`
- 只有少数指向实际的处理函数`__irq_usr(用户模式中断)`和`__irq_svc(管理模式中断)`，右边显示了中断号`@0~f`

**分发逻辑**
- 系统会检查SPSR寄存器来判断中断发生时CPU的工作模式
- 用户模式（USR）被中断，调用`__irq_usr`
- 内核模式（SVC）被中断，调用`__irq_svc`
- 这两个函数的处理流程实际上是相同的

**右侧-向量存根（Vector stubs）**
- 代码被复制到地址 `0xffff10000`，以便在向量中使用分支指令而不是跳转指令
- **重要限制**：代码不能超过一页大小
- 使用分支指令而不是跳转指令以提高效率
- 定义向量存根的宏，用于生成标准化的中断处理入口代码
```s
.macro vector_stub, name, mode, correction=0
.align 5
```

## 3 汇编到C语言的转换

### 3.1 模式处理函数的统一流程

**模式处理函数**：`__irq_usr`和`__irq_svc`这两个函数虽然针对不同模式，但处理流程完全相同：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/c08c24cc6c03c24640efa6cdb1ee200b.png)
**处理流程**：
```txt
保存现场(usr_entry) → 调用中断处理器(irq_handler) → 恢复现场(ret_to_user_from_irq)
```

### 3.2 关键转换：irq_handler宏

这是从**汇编转向C语言处理的关键点：**
```s
.macro  irq_handler
#ifdef CONFIG_GENERIC_IRQ_MULTI_HANDLER
    ldr r1, =handle_arch_irq    /* 加载C函数地址 */
    mov r0, sp                  /* 传递堆栈指针作为参数 */
    badr    lr, 9997f          /* 设置返回地址 */
    ldr pc, [r1]               /* 跳转到C函数 */
#else
    arch_irq_handler_default
#endif
9997:
.endm
```
- 该宏将控制权从汇编交给C函数的`handle_arch_irq`
- `handle_arch_irq`是一个函数指针，指向具体芯片的中断处理函数

### 3.3 handle_arch_irq的设置机制

**设置函数**：`set_handle_irq`（位于`内核源码/kernel/irq/handle.c +214`）
![image|400](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/848a24fd5252ec588a1973e6e7f975c9.png)

**重要概念**：不同芯片会设置自己特定的irq处理函数

### 3.4 S3C2440的具体实现

**对于S3C2440处理器的handle_arch_irq处理过程**：实际的处理函数是s3c24xx_handle_irq
- 所在路径：`内核源码/drivers/irqchip/irq-s3c24xx.c`
  ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7b7192e9ec83fcb7860a4b45abb4374e.png)

## 4 内核对中断的处理框架

### 4.1 基本设计思想

想象CPU连接了一个32位的中断控制器：
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/1be56738d964b2f49cc516174097476e.png)

**问题**：如何管理32种不同的中断及其处理函数？

**解决方案**：创建一个指针数组，每一项对应一个中断，**数组里面保存着中断处理函数指针**，这个数组的元素内核用`irq_desc`结构体表示

### 4.2 核心数据结构体：irq_desc

内核使用`irq_desc`结构体数组来管理所有中断：
![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/10d991e226d9c7a3928a71a87ce60a0a.png)

**关键成员理解**：
- **`action`链表**（`struct irqaction`）：
    - 链表中的`handler`是**我们编写的具体业务处理函数(专注于业务，不需要关注芯片相关的清除、使能等操作)**
    - 专注于业务逻辑，无需关注芯片底层操作
- **`handle_irq`**：
    - 负责调用action链表中的handler
    - **清理中断**（芯片相关操作），也就是调用`struct irq_desc`结构体中的`irq_data`里面的`chip`中的`*irq_ack`，调用链如下图
      ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/5cfbf549c0b175200d679c172d5cc137.png)

### 4.3 handler_arch_irq的通用处理流程

无论什么芯片，基本流程都是：
1. **读取中断控制器寄存器** → 获得硬件中断号（hwirq）
2. **转换中断号** → 将hwirq转换为虚拟中断号（virq）
3. **调用中断处理函数** → 执行`irq_desc[virq].handle_irq`

## 5 实际应用场景举例

### 5.1 场景1：共享中断处理

**硬件配置**：在0号中断上加一个或门芯片
![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/bb7178f7ff83852a17d156e9018ba7db.png)
- 一个引脚接网卡net，一个引脚接摄像头cam
- 当net或cam有数据时会产生中断，即共享中断的
- net的中断是irq_net，cam的中断是irq_cam，也就是在`S3C2410_IRQ(0)`这个irq_desc的action链表有两个irqaction，如下图
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/839dc45314a0cbabdd8176985a826277.png)
**当发生0号中断的时候，是如何处理的呢？**
1. `handle_irq`遍历action链表中的所有handler
2. **依次执行每个handler**（irq_net和irq_cam）
3. 每个handler内部判断是否为自己的设备产生的中断
4. 不是自己的中断就直接返回，是的话就执行相应处理

### 5.2 场景2：级联中断处理

**硬件配置**：4号中断连接子中断控制器（可能产生外部中断4，外部中断5等，和共享中断是类似的）
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/6f481dcec110c7866cc9a2d554e12f69.png)
**中断触发流程**：
- 当发生外部中断4、5、6、7任意一个时，`sub interrupt controller(子中断控制器)`都会向上一级的中断控制器`interrupt controller`发出信号，上一级的中断控制器就会向CPU发出中断
- CPU会读取某个寄存器，就会发现是4号中断，即 **上面4个外部中断发出信号时，CPU都会找到是4号中断**

**两种处理方法**：
**方法1（一般方法）**：
- 对于外部中断4、5、6、7任意一个，都可以写对应的中断处理函数
- 通过`*action`将它们链接在一起，最后都调用一遍确定是哪个外部中断

**方法2（智能分发）**：
- 既然它们都有`sub interrupt controller(子中断控制器)`了，可以让子中断控制器和一个寄存器`EINTPEND`来绑定
- 最后让4号中断对应的`irq_desc`中的`handler_irq`指向另外一个分发函数`s3c_irq_demux`，其工作如下
  ![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/42823c2f4ead6c765babeead4d39697a.png)
	- 读取Reg(EINTPEND)进一步分辨中断，得到`hwirq'`
	- 根据`hwirq'`得到`virq`
	- 调用`irq_desc[virq].handle_irq`处理子中断

### 5.3 具体实例：按键中断处理

**场景**：EINT5接了一个按键设备，中断处理函数是`irq_key` 如下图所示
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/df80b3e3c8e92e1a736ec1ed9058ce0e.png)

 ![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/3e65c9b7ea69ba0d75a4436ee237a6a4.png)
**完整处理流程**：
- 注册中断时，找到对应的中断号37
- 按键按下 → 信号传递：`子中断控制器 → 上级中断控制器 → CPU`
- CPU识别为4号中断，执行`irq_desc[4].handle_irq`
- `handle_irq`读取EINTPEND寄存器，确定是EINT5
- 然后得到EINT5对应的`hwirq'` -> `virq`，最后调用`irq_desc[virq].handle_irq`
- `irq_desc[virq].handle_irq`会将action里面的hanlder（即irq_key）取出来执行

## 6 源码分析：关键函数追踪

### 6.1 s3c24xx_handle_irq分析

**文件位置**：`内核源码/drivers/irqchip/irq-s3c24xx.c`
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/30c9dd6545976f9777b27b087cef6ef4.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/d4b5d7e53838d0ca88593908875c1bc8.png)

### 6.2 handle_domain_irq分析

**文件位置**：`内核源码/kernel/irq/irqdesc.c +654`
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/821a11772d1c416e45d0d49e6b4881f8.png)![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/064b350795c650d24030c6960c7b5334.png)

## 7 总结：两种中断处理模式

### 7.1 情况1：普通中断（无子中断）

当`irq_desc[virq].handle_irq`被调用时：
- 遍历`irq_desc[virq].action`链表中的每个handler并执行
- 使用`irq_desc[virq].irq_data.chip`中的函数清除中断标志

### 7.2 情况2：级联中断（有子中断）

当该中断由子中断控制器产生时：
- 读取sub int controller寄存器，得到hwirq'
- 根据hwirq'得到对应的virq
- 递归调用`irq_desc[virq].handle_irq`处理子中断

## 8 要点回顾

1. **硬件到软件的转换**：异常向量表 → 汇编处理 → C语言函数
2. **统一的管理框架**：irq_desc数组管理所有中断
3. **灵活的处理机制**：支持共享中断和级联中断
4. **分层设计思想**：底层处理芯片相关操作，上层专注业务逻辑
5. **可扩展性**：不同芯片可以注册自己的处理函数

通过理解这个框架，我们就能明白Linux内核是如何优雅地处理各种复杂的中断场景的。