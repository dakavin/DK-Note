# Daily Plan - 2025-06-28

## 1 基本信息

- 日期:: 2025-06-28
- 星期:: 星期六
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

- 活动:: 10:00-10:30-起床、洗漱-日常生活-30-
- 活动:: 10:30-11:18-GPIO和Pinctrl【训练营】-学习-48-1、复习整理一遍[[../../../../06-🐧 Linux系统/11-🚗 Linux驱动子系统详解(重点)/2_GPIO子系统/1_GPIO基础概念与硬件原理]]和[[../../../../06-🐧 Linux系统/11-🚗 Linux驱动子系统详解(重点)/2_GPIO子系统/2_GPIO编程接口与实践]]
- 活动:: 11:18-15:18-午饭、午休-日常生活-240-
- 活动:: 15:18-16:21-GPIO和Pinctrl【训练营】-学习-63-2、学习GPIO相关深入的内容
- 活动:: 16:21-16:45-休息-日常生活-24-
- 活动:: 16:45-17:52-GPIO和Pinctrl【训练营】-学习-67-3、继续学习GPIO相关深入的内容
- 活动:: 17:52-21:19-买菜、晚饭、休息-日常生活-207-
- 活动:: 21:19-22:19-GPIO和Pinctrl【训练营】-学习-60-4、优化pinctrl笔记
- 活动:: 22:19-22:40-洗澡、休息-日常生活-21-
- 活动:: 22:40-00:33-GPIO和Pinctrl【训练营】-学习-113-5、学习pinctrl基本定义和相关格式

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

