# Daily Plan - 2025-05-29

## 1 基本信息

- 日期:: 2025-05-29
- 星期:: 星期四
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

- 活动:: 07:00-07:30-起床洗漱-日常生活-30-
- 活动:: 07:30-08:12-送宝上班-日常生活-42-
- 活动:: 08:12-15:30-日常数据可视化（obsidian）-学习-438-完成可视化
- 活动:: 15:30-16:00-跳绳-运动-30-跳绳、拉伸
- 活动:: 16:00-16:33-休息-日常生活-33-吃西瓜、花生
- 活动:: 16:33-18:22-Linux内核-学习-109-没学一会，又因为vscode输入法无法输入中文打断了，搞了好久
- 活动:: 18:22-21:00-接宝下班，买菜做饭吃饭-日常生活-158-
- 活动:: 21:00-21:30-眯一会-日常生活-30-一天没有休息<br>睡一会
- 活动:: 21:30-22:09-理发-日常生活-39-没有理光，差评
- 活动:: 22:09-22:43-洗澡-日常生活-36-
- 活动:: 22:43-24:11-Linux内核-学习-88-1、处理apt的问题（用了ARM的源格式）<br>2、删除snap<br>3、安装deb的vscode适配中文输出

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

