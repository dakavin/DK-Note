# Daily Plan - 2025-06-05

## 1 基本信息

- 日期:: 2025-06-05
- 星期:: 星期四
- 天气:: ☁️
- 心情:: 还行

## 2 活动数据(自行记录)
活动记录格式：`活动:: 开始时间-结束时间-活动内容-类别-时长(分钟)-备注`
类别参考：
- 学习：技术学习、阅读、课程等
- 工作：工作相关任务
- 日常生活：吃饭、休息、家务等
- 运动：健身、跑步、瑜伽等
- 娱乐：游戏、看剧、音乐等
- 社交：聚会、聊天、会面等

- 活动:: 06:30-06:59-起床、洗漱、准备早餐-日常生活-29-
- 活动:: 06:59-07:18-Linux基础与应用开发，补充内容-学习-19-1、补充NFS，还剩下交叉编译和串口内容
- 活动:: 07:18-08:08-送宝上班、吃早餐-日常生活-50-
- 活动:: 08:08-09:12-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/1_设备树的引进和体验]]-学习-64-1、跟着过一遍总线平台设备模型和设备树模型的写法<br>2、而外知道了通过device获取设备树节点的办法<br>3、探讨一下设备树
- 活动:: 09:12-09:40-休息-日常生活-28-
- 活动:: 09:40-10:54-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/2_设备树的规范（dts和dtb）]]-学习-74-1、完成对dts格式的复习<br>2、详细对dtb格式进行学习
- 活动:: 10:54-11:30-休息、煮饭-日常生活-36-
- 活动:: 11:30-12:09-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/3_内核对设备树的处理]]-学习-39-1、整理全篇笔记
- 活动:: 12:09-14:05-午饭、午休-日常生活-116-
- 活动:: 14:05-16:17-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/3_内核对设备树的处理]]-学习-132-2、完成如下学习<br>①内核对DTB的简单处理<br>②内核对DTB平台信息的处理<br>③内核对DTB运行配置信息处理<br>④内核对常规信息转化为device_node
- 活动:: 16:17-16:56-休息-日常生活-39-
- 活动:: 16:56-18:00-接宝下班、买菜-日常生活-64-
- 活动:: 18:00-18:44-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/3_内核对设备树的处理]]-学习-44-3、对于各个函数流程，自己分析一遍
- 活动:: 18:44-21:38-晚饭、晚休-日常生活-174-
- 活动:: 21:38-23:18-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/3_内核对设备树的处理]]-学习-100-3、完成如下学习<br>①device_node转换为platform_device<br>②注册platform_device和platform_driver时是如何match的<br>③常用的of函数（针对DTB、device_node、platform_device）<br>④常见的调试手段
- 活动:: 23:18-00:00-洗澡-日常生活-42-

## 3 详细活动记录

### 3.1 今日活动概览

```dataview
TABLE WITHOUT ID
  split(活动, "-")[0] as "开始时间",
  split(活动, "-")[1] as "结束时间", 
  split(活动, "-")[2] as "活动内容",
  split(活动, "-")[3] as "类别",
  (floor(number(split(活动, "-")[4]) / 60) + "小时" + (number(split(活动, "-")[4]) % 60) + "分钟") as "时长",
  split(活动, "-")[5] as "备注"
WHERE file.name = this.file.name
FLATTEN 活动
WHERE 活动 AND contains(活动, "-") AND length(split(活动, "-")) >= 5 AND contains(split(活动,"-")[0],":")
SORT split(活动, "-")[0] ASC
```

### 3.2 今日学习活动

```dataview
TABLE WITHOUT ID
  key as "学习内容",
  (floor(sum(map(rows, (r) => number(split(r.活动, "-")[4]))) / 60) + "小时" + (sum(map(rows, (r) => number(split(r.活动, "-")[4]))) % 60) + "分钟") as "总时长",
  join(map(filter(rows, (r) => split(r.活动, "-")[5] != ""), (r) => split(r.活动, "-")[5]), "<br>") as "备注"
WHERE file.name = this.file.name
FLATTEN 活动
WHERE 活动 AND contains(split(活动, "-")[3], "学习") AND length(split(活动, "-")) >= 5
GROUP BY split(活动, "-")[2] as key
SORT key ASC

```

**今日学习总时长：**

```dataview
LIST WITHOUT ID
"总计：" + floor(sum(map(filter(活动, (x) => contains(split(x, "-")[3], "学习")), (r) => number(split(r, "-")[4]))) / 60) + "小时" + (sum(map(filter(活动, (x) => contains(split(x, "-")[3], "学习")), (r) => number(split(r, "-")[4]))) % 60) + "分钟"
WHERE file.name = this.file.name
```

### 3.3 今日时间分配统计

```dataview
TABLE WITHOUT ID
  key as "活动类别",
  length(rows) as "活动次数",
  (floor(sum(map(rows, (r) => number(split(r.活动, "-")[4]))) / 60) + "小时" + (sum(map(rows, (r) => number(split(r.活动, "-")[4]))) % 60) + "分钟") as "总时长",
  round(sum(map(rows, (r) => number(split(r.活动, "-")[4]))) / 60, 1) + "小时" as "总时长(小数)"
WHERE file.name = this.file.name
FLATTEN 活动
WHERE 活动 AND contains(活动, "-") AND length(split(活动, "-")) >= 5
GROUP BY split(活动, "-")[3] as key
SORT sum(map(rows, (r) => number(split(r.活动, "-")[4]))) DESC
```

## 4 今日总结

### 4.1 学习成果记录

韦东山老师设备树的课程已经完成3课
- 前面两课可以快速过了
- 关键1：dtb的格式
- 关键2：内核是如何利用dtb的
	1. head.S文件获取一些配置信息即根节点的compatible和machine_desc进行比较，选择合适的板卡
	2. 获取一些运行时的配置，如chosen、size、memory
	3. 将一些设备信息转换为device_node
	4. device_node转换为platform_device
	5. platform_device和platform_driver利用platform_bus_type的match进行匹配
	6. 常见的of函数
	7. 文件系统如何查看设备树

### 4.2 剩余代办事项

设备树还没有看完！

### 4.3 反思和改进

今天也学够了8小时，主要是内核如何利用dtb这里花费时间比较久，也比较难，需要多看看

明天加油把设备树看完！
