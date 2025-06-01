# Daily Plan - 2025-05-30

## 1 基本信息

- 日期:: 2025-05-30
- 星期:: 星期五
- 天气:: 阴天
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

- 活动:: 07:15-08:00-起床-日常生活-45-起床、洗漱、煮粽子、烧水
- 活动:: 08:00-08:50-[[../../../../06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/4_Linux驱动开发实战/1_Linux驱动基础知识(重点)/3_Linux内核模块实验]]-学习-50-1、内核设备传参<br>2、了解通用的三种抽象设备类型
- 活动:: 08:50-09:09-吃早餐-日常生活-19-
- 活动:: 09:09-10:03-[[../../../../06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/4_Linux驱动开发实战/1_Linux驱动基础知识(重点)/4_📕字符设备驱动]]-学习-54-3、字符设备是如何抽象的<br>4、理解设备号、cdev、设备节点
- 活动:: 10:03-11:00-休息-日常生活-57-
- 活动:: 11:00-12:15-[[../../../../06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/4_Linux驱动开发实战/1_Linux驱动基础知识(重点)/4_📕字符设备驱动]]-学习-75-5、理解inode、file、file_operations<br>6、贯通“号档节点一文操”
- 活动:: 12:15-16:06-休息-日常生活-231-午饭、**午休过头了啊**
- 活动:: 16:06-17:11-[[../../../../06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/4_Linux驱动开发实战/1_Linux驱动基础知识(重点)/4_📕字符设备驱动]]-学习-65-7、字符设备驱动程序框架
- 活动:: 17:11-18:33-买菜-日常生活-82-
- 活动:: 18:33-19:03-[[../../../../06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/4_Linux驱动开发实战/1_Linux驱动基础知识(重点)/4_📕字符设备驱动]]-学习-30-8、补充剩余的mknod的使用
- 活动:: 19:03-20:05-晚餐-日常生活-62-
- 活动:: 20:05-21:36-[[../../../../06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/4_Linux驱动开发实战/1_Linux驱动基础知识(重点)/4_📕字符设备驱动]]-学习-91-9、理解open函数的流程<br>10、搭建字符设备驱动程序代码框架
- 活动:: 21:36-22:34-休息、洗澡、泡茶-日常生活-58-
- 活动:: 22:34-22:52-配置netcup账户-学习-18-还没有付款，先配置了
- 活动:: 22:34-23:59-[[../../../../06-🐧 Linux系统/04-🔌 驱动开发/02-💾 Lubancat-RK3568/4_Linux驱动开发实战/1_Linux驱动基础知识(重点)/4_📕字符设备驱动]]-学习-85-解决vscode的编译问题去了

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

1、完成内核模块实验
2、通过学习字符设备驱动，对于设备驱动框架有了一个完整的认识

### 4.2 剩余代办事项

1、明天要加快一下进度，完成字符设备驱动的代码实验！！！
2、然后争取快速学完5、6、7节课

### 4.3 反思和改进

1、午休太久啦！！！
2、晚上剩余时间又在纠结vscode的检索问题，也浪费了很多时间，不值得，没必要为了完美的IDE体验搞得太复杂
