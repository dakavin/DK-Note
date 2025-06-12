# Daily Plan - 2025-06-06

## 1 基本信息

- 日期:: 2025-06-06
- 星期:: 星期五
- 天气:: 
- 心情::

## 2 活动数据(自行记录)
活动记录格式：`活动:: 开始时间-结束时间-活动内容-类别-时长(分钟)-备注`
类别参考：
- 学习：技术学习、阅读、课程等
- 工作：工作相关任务
- 日常生活：吃饭、休息、家务等
- 运动：健身、跑步、瑜伽等
- 娱乐：游戏、看剧、音乐等
- 社交：聚会、聊天、会面等

- 活动:: 08:08-09:12-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/1_设备树的引进和体验]]-学习-64-1、跟着过一遍总线平台设备模型和设备树模型的写法<br>2、而外知道了通过device获取设备树节点的办法<br>3、探讨一下设备树
- 活动:: 09:12-09:40-休息-日常生活-28-
- 活动:: 09:40-10:54-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/2_设备树的规范（dts和dtb）]]-学习-74-1、完成对dts格式的复习<br>2、详细对dtb格式进行学习
- 活动:: 10:54-11:30-休息、煮饭-日常生活-36-
- 活动:: 11:30-12:09-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/3_内核对设备树的处理]]-学习-39-1、整理全篇笔记
- 活动:: 12:09-13:00-午饭、午休-日常生活-51-
- 活动:: 13:00-15:17-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/3_内核对设备树的处理]]-学习-137-2、完成如下学习<br>①内核对DTB的简单处理<br>②内核对DTB平台信息的处理<br>③内核对DTB运行配置信息处理<br>④内核对常规信息转化为device_node
- 活动:: 15:17-15:40-休息-日常生活-23-
- 活动:: 15:40-17:17-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/4_uboot对设备树的支持]]-学习-97-
- 活动:: 17:17-23:00-买菜、晚饭、休息-日常生活-343-

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

### 4.2 剩余代办事项

### 4.3 反思和改进

