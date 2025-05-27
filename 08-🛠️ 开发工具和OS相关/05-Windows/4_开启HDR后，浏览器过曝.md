---
文章标题: "[[4_开启HDR后，浏览器过曝]]" 
文章作者: Dakkk
文章概要: |
  文章阐述了系统开启HDR模式后，浏览器截图出现过曝或泛白的问题。核心原因在于Chrome与Windows的颜色管理不兼容。解决办法是访问`chrome://flags/`，调整`force color profile`选项匹配显示器颜色设置，重启Chrome即可。
tags:
- "HDR"
- "浏览器"
- "颜色管理"
- "Chrome"
- "Windows"
- "截图"
- "过曝"
- "泛白"
相关文章:
- "[[1_MinGW编译器的安装和配置]]"
文章分类: "🛠️ 开发工具和OS相关"
文章路径: "08-🛠️ 开发工具和OS相关/05-Windows/4_开启HDR后，浏览器过曝.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2024-08-08 22:28:40
修改时间: 2025-05-28 01:03:09
---

浏览器截图出现泛白或过曝的主要原因是系统开启了HDR模式导致的颜色管理问题。具体表现为`[1][2][3][4][5]`：

1. 支持HDR的显示器开启HDR模式后，找一个网页HDR视频播放，按下截图键会出现严重的过饱和和偏色情况。

2. Chrome浏览器会默认匹配系统的颜色管理，但是由于截图瞬间颜色管理出错，导致颜色显示不对。

3. 这个问题可能是由于Chrome浏览器和Windows系统的颜色管理不兼容所导致的。当Windows显示开启HDR后，系统的颜色管理发生变化，而Chrome浏览器无法正确识别和处理这些变化。

解决方法是在Chrome地址栏输入`chrome://flags/`，搜索`force color profile`选项，选择与显示器相匹配的颜色管理设置，如Display P3、sRGB或Adobe RGB，然后重启Chrome即可[3][4][5]。

总之，浏览器截图泛白是由于系统HDR模式与浏览器颜色管理不兼容导致的。通过调整Chrome的颜色管理设置可以解决这个问题。

Citations:
[1] https://zhidao.baidu.com/question/336097210542659325.html
[2] https://g.nga.cn/read.php?tid=24941258
[3] https://www.cnblogs.com/Jankin-Wen/p/17738219.html
[4] https://blog.csdn.net/Athus_c/article/details/106494715
[5] https://developer.baidu.com/article/details/2874573