# Daily Plan - 2025-06-08

## 1 基本信息

- 日期:: 2025-06-08
- 星期:: 星期日
- 天气:: ☀️☀️☀️
- 心情:: 还阔以

## 2 活动数据(自行记录)
活动记录格式：`活动:: 开始时间-结束时间-活动内容-类别-时长(分钟)-备注`
类别参考：
- 学习：技术学习、阅读、课程等
- 工作：工作相关任务
- 日常生活：吃饭、休息、家务等
- 运动：健身、跑步、瑜伽等
- 娱乐：游戏、看剧、音乐等
- 社交：聚会、聊天、会面等

- 活动:: 07:36-07:52-起床、洗漱、准备早餐-日常生活-16-
- 活动:: 07:52-09:43-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/5_中断系统中的设备树]]-学习-111-1、完成内核对中断处理的框架和代码流程的学习
- 活动:: 09:43-10:20-休息-日常生活-37-
- 活动:: 10:20-12:11-[[3_中断号的演变与irq_domain]]-学习-111-2、学习hwirq和virq在传统和irq_domain两种情况下是如何绑定的
- 活动:: 12:11-14:00-午休-日常生活-109-
- 活动:: 14:00-15:21-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/5_中断系统中的设备树]]-学习-81-3、学习设备树是如何描述中断控制器和中断设备的<br>4、学习使用设备树来描述具体的按键中断
- 活动:: 15:21-15:42-休息-日常生活-21-
- 活动:: 15:42-16:36-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/5_中断系统中的设备树]]-学习-54-5、学习内核对设备树中断信息的处理（文本内容填写）
- 活动:: 16:36-17:24-休息-日常生活-48-
- 活动:: 17:24-17:45-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/5_中断系统中的设备树]]-学习-21-6、学习内核对设备树中断信息的处理（关键的irq_domain）
- 活动:: 17:45-18:47-晚饭-日常生活-62-
- 活动:: 18:47-20:34-睡觉-日常生活-107-
- 活动:: 20:34-21:50-设备树已学知识总结-学习-76-
- 活动:: 21:50-22:20-休息-日常生活-30-
- 活动:: 22:20-23:30-设备树已学知识总结-学习-70-

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

