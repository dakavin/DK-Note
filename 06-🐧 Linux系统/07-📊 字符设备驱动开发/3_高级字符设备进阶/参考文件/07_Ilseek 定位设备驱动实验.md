假如现在有这样一个场景， 将两个字符串依次进行写入， 并对写入完成的字符串进行读取， 如果仍采用之前的方式， 第二次的写入值会覆盖第一次写入值， 那要如何来实现上述功能呢？ 这就要轮到 llseek 出场了 。

在应用程序中使用 lseek 函数进行读写位置的调整， 该函数的具体使用说明如下所示： lseek 函数 a
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/60c91a26cbcd0e2298742e0791930f1c.png)


函数使用示例：

把文件位置指针设置为 5：

`lseek(fd,5,SEEK_SET);`

把文件位置设置成文件末尾：

`lseek(fd,0,SEEK_END);`

确定当前的文件位置：

`lseek(fd,0,SEEK_CUR);`