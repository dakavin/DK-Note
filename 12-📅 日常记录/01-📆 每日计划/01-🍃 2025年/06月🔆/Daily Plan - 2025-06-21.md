# Daily Plan - 2025-06-21

## 1 基本信息

- 日期:: 2025-06-21
- 星期:: 星期六
- 天气:: 🌥️
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

- 活动:: 07:00-08:15-早起事项-日常生活-75-起床、洗漱、送宝去地铁站、吃早餐、泡茶、上厕所
- 活动:: 08:15-09:00-复习【训练营】-学习-45-1、内核模块的复习
- 活动:: 09:00-09:30-复习【训练营】-学习-30-2、设备模型的复习
- 活动:: 09:30-13:00-休息-日常生活-210-补觉
- 活动:: 13:00-15:35-字符设备【训练营】-学习-155-1、完成设备分类和字符设备抽象设计导图<br>2、完成设备设备相关结构体和设备节点创建导图
- 活动:: 15:35-16:00-休息-日常生活-25-
- 活动:: 16:00-17:03-字符设备【训练营】-学习-63-3、完成字符设备开发流程
- 活动:: 17:03-21:00-接宝、买菜、做饭、晚饭-日常生活-237-
- 活动:: 21:00-21:35-晚休-日常生活-35-
- 活动:: 21:35-22:09-字符设备【训练营】-学习-34-4、完成字符设备驱动和open函数解析导图
- 活动:: 22:09-22:33-洗澡-日常生活-24-
- 活动:: 22:33-22:48-字符设备【训练营】-学习-15-5、完成字符设备技巧导图
- 活动:: 22:48-23:08-休息、上厕所-日常生活-20-
- 活动:: 23:08-00:32-字符设备【训练营】-学习-84-5、完成Linux错误机制导图<br>6、完成笔记汇总
- 活动:: 00:32-01:00-上厕所-日常生活-28-
- 活动:: 01:00-01:46-Win配置-娱乐-46-共享Win的笔记库给Ubuntu

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

