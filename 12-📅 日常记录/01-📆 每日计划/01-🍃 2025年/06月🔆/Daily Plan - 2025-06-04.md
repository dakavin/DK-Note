# Daily Plan - 2025-06-04

## 1 基本信息

- 日期:: 2025-06-04
- 星期:: 星期三
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

- 活动:: 06:30-07:00-起床、洗漱、准备早餐-日常生活-30-
- 活动:: 07:00-07:49-吃早餐、看小说和论坛-日常生活-49-
- 活动:: 07:49-08:29-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/03-📊 字符设备驱动模型/1_字符设备驱动模型基础(Lubancat)/8_Linux设备树]]-学习-40-1、复习一遍昨天所学内容<br>2、完成剩余在驱动获取DTS属性的实验
- 活动:: 08:29-08:59-上厕所-日常生活-30-
- 活动:: 08:59-11:26-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/03-📊 字符设备驱动模型/1_字符设备驱动模型基础(Lubancat)/9_Linux设备树_LED灯实验]]-学习-147-完成实验，并解决设备树reg属性读取失败的问题
- 活动:: 11:26-11:44-休息-日常生活-18-
- 活动:: 11:44-12:10-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/03-📊 字符设备驱动模型/1_字符设备驱动模型基础(Lubancat)/10_设备树插件(Device Tree Overlays)]]-学习-26-1、完成设备树插件的了解和实验
- 活动:: 12:10-12:26-休息-日常生活-16-
- 活动:: 12:26-12:38-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/1_设备树的引进和体验]]-学习-12-1、看视频回顾一下字符设备驱动程序的三种写法
- 活动:: 12:38-13:40-午饭-日常生活-62-
- 活动:: 13:40-16:33-午休-日常生活-173-
- 活动:: 16:33-17:49-[[../../../../06-🐧 Linux系统/07-🚗 Linux驱动开发核心(重要!!!)/05-🚗 Linux驱动相关子系统 (重点)/0——/2_Linux设备树详解（韦东山）/1_设备树的引进和体验]]-学习-76-2、回顾一下字符设备驱动的传统写法和编译测试
- 活动:: 17:49-18:38-Linux基础与应用开发，补充内容-学习-49-
- 活动:: 18:38-23:37-晚饭、晚休、洗澡-日常生活-299-状态不好，休息一下
- 活动:: 23:37-00:06-Linux基础与应用开发，补充内容-学习-29-

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

