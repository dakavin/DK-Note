# Daily Plan - 2025-06-02

## 1 基本信息

- 日期:: 2025-06-02
- 星期:: 星期一
- 天气:: 🌥️
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

- 活动:: 06:29-07:04-起床、洗漱、刷牙、弄早餐-日常生活-35-
- 活动:: 07:04-07:41-[[6_Linux的设备驱动模型]]-学习-37-1、回顾昨天学习的总线<br>2、详细学习一下设备和驱动
- 活动:: 07:41-08:17-早餐-日常生活-36-
- 活动:: 08:17-09:05-[[6_Linux的设备驱动模型]]-学习-48-3、详细学习attribute这个结构体
- 活动:: 09:05-09:55-买菜、上厕所-日常生活-50-
- 活动:: 09:55-11:15-[[6_Linux的设备驱动模型]]-学习-80-4、完成简单的设备驱动模式实验，新建总线、设备、驱动模块，并配置其相关属性
- 活动:: 11:15-11:21-[[7_平台设备驱动]]-学习-6-1、简单预习一下目录
- 活动:: 11:21-13:00-午饭、做菜-日常生活-99-
- 活动:: 13:00-14:54-午休-日常生活-114-
- 活动:: 14:54-15:18-[[7_平台设备驱动]]-学习-24-2、了解一下为什么需要平台设备以及平台设备的部分结构体
- 活动:: 15:18-16:22-送宝上班+休息-日常生活-64-
- 活动:: 16:22-17:33-[[7_平台设备驱动]]-学习-71-3、学习平台驱动和平台总线的结构体、相关注册和注销方式等<br>4、学习平台总线的匹配机制（id_table和name）
- 活动:: 17:33-18:14-跳绳-运动-41-今天多一组，总共3组，加油
- 活动:: 18:14-19:20-晚饭、晚休-日常生活-66-
- 活动:: 19:20-21:09-[[7_平台设备驱动]]-学习-109-5、完成平台实验代码（编写平台设备和平台驱动代码）
- 活动:: 21:09-21:28-Linux驱动基础知识（重点）-学习-19-1、建立模板，待会分析和填写
- 活动:: 21:28-22:01-洗澡-日常生活-33-
- 活动:: 22:01-23:32-Linux驱动基础知识（重点）-学习-91-1、思维导图总结原始架构的结构体、相关API和实验流程-Linux驱动基础知识（重点）-学习-91-1、思维导图总结原始架构的结构体、相关API和实验流程

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

