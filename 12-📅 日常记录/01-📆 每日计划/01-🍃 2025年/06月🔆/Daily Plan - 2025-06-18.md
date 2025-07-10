# Daily Plan - 2025-06-18

## 1 基本信息

- 日期:: 2025-06-18
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

- 活动:: 11:00-12:00-起床、洗漱、午饭-日常生活-60-
- 活动:: 12:00-13:36-设备相关模型【训练营】-学习-96-1、学习[[1_Linux设备模型基础/3_kobject实践指南]]<br>2、学习[[1_Linux设备模型基础/4_kset实践指南]]
- 活动:: 13:36-14:03-上厕所、休息-日常生活-27-
- 活动:: 14:03-15:08-设备相关模型【训练营】-学习-65-3、学习[[1_Linux设备模型基础/5_属性文件系统详解]]
- 活动:: 15:08-15:35-设备相关模型【训练营】-学习-27-4、学习[[1_Linux设备模型基础/6_SysFS设备驱动管理]]
- 活动:: 15:35-15:45-休息-日常生活-10-
- 活动:: 15:45-16:56-设备相关模型【训练营】-日常生活-71-5、复习[[1_Linux设备模型基础/7_总线详解与实践]]
- 活动:: 16:56-19:36-晚饭、晚休-日常生活-160-
- 活动:: 19:36-21:33-设备相关模型【训练营】-学习-117-6、重新过一编之前的内容<br>7、复习[[1_Linux设备模型基础/8_设备详解与实践]]<br>8、复习[[1_Linux设备模型基础/9_驱动详解与实践]]
- 活动:: 21:33-22:15-休息-日常生活-42-
- 活动:: 22:15-23:52-设备相关模型【训练营】-学习-97-9、完成总线、设备、驱动相关实验
- 活动:: 23:52-00:30-设备相关模型【训练营】-学习-38-10、研究思维导图的绘制

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

