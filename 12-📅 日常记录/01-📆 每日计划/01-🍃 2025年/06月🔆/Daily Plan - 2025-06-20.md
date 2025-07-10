# Daily Plan - 2025-06-20

## 1 基本信息

- 日期:: 2025-06-20
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

- 活动:: 08:30-09:00-起床、洗漱、吃早餐-日常生活-30-
- 活动:: 09:00-09:40-设备相关模型【训练营】-学习-40-1、复习一遍昨天的流程图<br>2、完成驱动注册机制流程图
- 活动:: 09:40-11:00-内核模块【训练营】-学习-80-1、完成内核源码目录结构图<br>2、完成HelloModule模块实验导图
- 活动:: 11:00-11:25-休息-日常生活-25-
- 活动:: 11:25-13:10-内核模块【训练营】-学习-105-3、系统调用导图<br>4、完成ko文件接口导图
- 活动:: 13:10-14:30-午饭、午休-日常生活-80-
- 活动:: 14:30-15:59-内核模块【训练营】-学习-89-5、完成内核模块相关命令导图<br>6、完成内核模块传参和符号共享导图<br>7、完成内核加载和卸载流程导图
- 活动:: 15:59-16:29-休息-日常生活-30-
- 活动:: 16:29-17:43-内核模块【训练营】-学习-74-8、完成module_init优先级导图<br>9、完成内核配置导图<br>10、整理成思维导图笔记
- 活动:: 17:43-17:54-平台设备模型【训练营】-学习-11-1、整理笔记框架
- 活动:: 17:54-18:48-休息-日常生活-54-
- 活动:: 18:48-20:48-平台设备模型【训练营】-学习-120-1、复习[[1_平台设备模型的引入]]<br>2、复习[[2_platform 总线模型概述]]<br>3、复习[[3_platform 总线注册与初始化]]<br>4、复习[[4_platform 设备注册与管理]]<br>5、复习[[5_platform 驱动注册与管理]]<br>6、复习[[6_platform 设备和驱动匹配机制]]<br>7、复习[[7_platform 设备资源获取与管理]]
- 活动:: 20:48-20:58-休息-日常生活-10- 
- 活动:: 20:58-21:22-字符设备【训练营】-学习-24-1、整理文章目录结构 
- 活动:: 21:22-23:27-吃烧烤、看电影-娱乐-125- 
- 活动:: 23:27-23:58-洗澡-日常生活-31- 
- 活动:: 23:58-00:58-字符设备【训练营】-学习-60-2、快速复习完字符设备相关内容
- 活动:: 00:58-01:48-内核互斥【训练营】-学习-50-1、整理部分笔记框架

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

