# Daily Plan - 2025-06-03

## 1 基本信息

- 日期:: 2025-06-03
- 星期:: 星期二
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

- 活动:: 09:40-10:11-起床、洗漱、准备早餐-日常生活-31-
- 活动:: 10:11-10:30-Linux驱动基础知识（重点）-学习-19-1、回顾复习一下原始架构
- 活动:: 10:30-11:15-吃早餐，上厕所-日常生活-45-
- 活动:: 11:15-12:22-Linux驱动基础知识（重点）-学习-67-2、复习一下设备驱动模型
- 活动:: 12:22-13:14-午饭-日常生活-52-
- 活动:: 13:14-14:15-午休-日常生活-61-
- 活动:: 14:15-15:40-Linux驱动基础知识（重点）-学习-85-3、复习一遍平台设备驱动
- 活动:: 15:40-16:05-休息-日常生活-25-
- 活动:: 16:05-17:52-[[../../../../06-🐧 Linux系统/10-📊 字符设备驱动开发/1_字符设备驱动模型基础(Lubancat)/8_Linux设备树]]-学习-107-1、学习设备树的简介和设备树的格式/框架
- 活动:: 17:52-19:35-晚饭-日常生活-103-
- 活动:: 19:35-20:35-晚休-日常生活-60-
- 活动:: 20:35-22:02-[[../../../../06-🐧 Linux系统/10-📊 字符设备驱动开发/1_字符设备驱动模型基础(Lubancat)/8_Linux设备树]]-学习-87-2、学习设备相关API（获取节点和获取节点属性）
- 活动:: 22:02-22:42-洗澡、休息-日常生活-40-
- 活动:: 22:42-23:56-[[../../../../06-🐧 Linux系统/10-📊 字符设备驱动开发/1_字符设备驱动模型基础(Lubancat)/8_Linux设备树]]-学习-74-3、学习剩余API（获取物理地址）<br>4、完成添加设备树节点实验，并cat获取相关属性

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

完成了设备树相关内容的学习，具体如下:
1. 设备树的概念
2. 设备树的格式
3. 设备树常见的属性
4. 设备树of函数
	- 获取节点的of函数
	- 获取节点属性的of函数
	- 获取节点物理地址的of函数
5. 完成了一个添加设备树节点的实验，并通过cat /proc/device-tree的方式查看属性

### 4.2 剩余代办事项

设备树最后的内容 `在驱动中获取节点属性` 的实验没有完成！！！

### 4.3 反思和改进

今天学习进度有点慢，浪费了很多时间，导致一章内容也没有学完

按道理，设备树之前稍微学习过了，所以应该要加快进度！！！