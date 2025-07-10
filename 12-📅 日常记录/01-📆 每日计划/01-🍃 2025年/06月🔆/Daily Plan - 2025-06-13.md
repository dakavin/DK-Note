# Daily Plan - 2025-06-13

## 1 基本信息

- 日期:: 2025-06-13
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

- 活动:: 07:20-07:44-起床、洗漱-日常生活-24-
- 活动:: 07:44-09:11-内核模块【训练营】-学习-87-1、处理系统调用问题
- 活动:: 09:11-09:33-休息、吃早餐-日常生活-22-
- 活动:: 09:33-11:12-内核模块【训练营】-学习-99-2、快速学习Linux热插拔机制和mdev使能；<br>3、快速学习内核事件通知机制和udev通信；<br>4、整理linux内核学习笔记
- 活动:: 11:12-12:19-琢磨cursor pro-娱乐-67-失败了，有空再弄吧
- 活动:: 12:19-13:01-内核模块【训练营】-学习-42-5、[[8_Linux内核配置系统详解(menuconfig)]]的学习，作为内核模块的补充
- 活动:: 13:01-14:00-午饭-日常生活-59-
- 活动:: 14:00-14:30-午休-日常生活-30-
- 活动:: 14:30-16:36-中断【训练营】-学习-126-1、完成笔记内容复制和归档整理
- 活动:: 16:36-17:05-拉伸运动-运动-29-
- 活动:: 17:05-18:35-中断【训练营】-学习-90-2、学习[[../../../../06-🐧 Linux系统/09-🔍 中断及异常处理/参考内容/训练营/1_中断概念]]；<br>3、学习[[../../../../06-🐧 Linux系统/09-🔍 中断及异常处理/参考内容/训练营/2_异常概念]]；<br>4、学习[[../../../../06-🐧 Linux系统/09-🔍 中断及异常处理/参考内容/训练营/4_获取中断号]]
- 活动:: 18:35-21:35-晚饭、晚休-日常生活-180-
- 活动:: 21:35-22:57-中断【训练营】-学习-82-5、学习[[../../../../06-🐧 Linux系统/09-🔍 中断及异常处理/参考内容/训练营/10_中断子系统框架]]


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

