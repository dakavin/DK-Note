# Daily Plan - 2025-06-23

## 1 基本信息

- 日期:: 2025-06-23
- 星期:: 星期一
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

- 活动:: 09:30-10:10-起床、洗漱、泡茶-日常生活-40-
- 活动:: 10:10-10:41-设备树【导图】-学习-31-1、完成设备树导图架构和设备树概述部分内容
- 活动:: 10:41-11:34-吃饭、休息-日常生活-53-
- 活动:: 11:34-12:52-设备树【导图】-学习-78-2、完成设备树概述内容；<br>3、完成设备树图解
- 活动:: 12:52-15:43-午休-日常生活-171-
- 活动:: 15:43-17:17-设备树【导图】-学习-94-4、完成CPU和时钟属性图解
- 活动:: 17:17-19:30-买菜、做饭、晚饭-日常生活-133-
- 活动:: 19:30-21:05-洗风扇-日常生活-95-
- 活动:: 21:05-22:09-设备树【导图】-学习-64-5、完成中断属性图解
- 活动:: 22:09-22:44-洗澡-日常生活-35-
- 活动:: 22:44-23:27-设备树【导图】-学习-43-6、完成gpio和pinctrl属性图解
- 活动:: 23:27-23:48-内核互斥【训练营】-学习-21-1、复习一遍1-4的内容
- 活动:: 23:48-23:58-休息-日常生活-10-
- 活动:: 23:58-00:23-内核互斥【训练营】-学习-25-2、复习一遍5-14的内容
- 活动:: 00:23-00:40-休息-日常生活-17-
- 活动:: 00:40-01:58-内核互斥【训练营】-学习-78-3、学习[[15_信号处理机制(软件中断_了解)]]<br>4、学习[[16_内敛汇编(了解)]]<br>5、学习[[17_死锁检测工具lockdep]]<br>6、学习[[18_面试题]]

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

