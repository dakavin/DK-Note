# Daily Plan - 2025-06-10

## 1 基本信息

- 日期:: 2025-06-10
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

- 活动:: 08:00-08:30-起床、洗漱-日常生活-30-
- 活动:: 08:30-09:30-设备树总结-学习-60-总结了以下内容<br>1、设备的引进；<br>2、设备树DTS和DTB的结构，相关转换；
- 活动:: 09:30-09:49-吃早餐、休息-日常生活-19-
- 活动:: 09:49-10:03-上厕所-日常生活-14-
- 活动:: 10:03-10:53-设备树总结-学习-50-3、内核对设备树的处理；<br>4、of函数和调试手段；<br>5、uboot对设备树的处理
- 活动:: 10:53-11:22-休息-日常生活-29-
- 活动:: 11:22-11:43-中断总结-学习-21-1、中断的引入、概念和处理流程；
- 活动:: 11:43-12:30-午饭-日常生活-47-
- 活动:: 12:30-12:54-午休-日常生活-24-
- 活动:: 12:54-13:40-中断总结-学习-46-2、Linux中断处理框架及代码流程；<br>3、中断号的演变和irq_domain
- 活动:: 13:40-14:19-设备树总结-学习-39-6、设备树中断属性、Linux对设备树中断的处理和驱动处理中断相关API；<br>7、设备树中的时钟和pinctrl描述，内核对其处理和相关API
- 活动:: 14:19-14:47-休息-日常生活-28-
- 活动:: 14:47-15:22-设备树【训练营】-学习-35-1、完成思维导图的绘制；<br>2、完成设备节点的创建和文件系统中的验证
- 活动:: 15:22-16:48-送宝上班，休息-日常生活-86-
- 活动:: 16:48-17:23-设备树【训练营】-学习-35-3、完成作业和问题的编写
- 活动:: 17:23-18:02-设备树【训练营】-学习-39-4、快速过一遍设备树的视频，过一遍笔记的重写方法
- 活动:: 18:02-20:00-晚饭、晚休-日常生活-118-
- 活动:: 20:00-21:26-笔记库-工作-86-处理笔记库超过100MB的PDF文档无法上传问题
- 活动:: 21:26-21:51-洗澡-日常生活-25-
- 活动:: 21:51-23:54-设备树【训练营】-学习-123-完成如下笔记的重写和复习<br>1、设备树简介<br>2、设备树常见属性<br>3、设备树中对CPU和时钟的描述

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

