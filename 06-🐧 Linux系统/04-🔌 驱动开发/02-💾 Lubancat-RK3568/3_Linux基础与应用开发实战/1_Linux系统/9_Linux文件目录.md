参考文档：[2_Linux基础命令](../../../../02-⚙️%20系统基础/01-💻%20命令行操作/2_Linux基础命令.md)

补充：根目录的内容如下，具体描述参考[2 根文件系统目录简介](../../2_镜像构建与部署/12_根文件系统的介绍.md#2%20根文件系统目录简介)

|目录|目录放置的内容|
|---|---|
|bin|存放系统命令的目录，如cat，cp，mkdir命令|
|boot|存放开机启动过程所需的内容，如开机管理程序grub2|
|dev|所有设备文件的目录（如声卡、硬盘、光驱）|
|etc|系统的主要配置文件|
|home|用户家目录数据的存放目录|
|lib|存放sbin和bin目录下命令所需的库文件|
|lib32/lib64|存放二进制函数库，支持32位/64位|
|lost+found|在EXT3/4系统中，当系统意外崩溃或意外关机时，会产生一些碎片文件在这个目录下面，系统启动fcsk工具会检查这个目录，并修复已损坏的文件。|
|media|用于挂载光盘，软盘和DVD等设备|
|mnt|同media作用一样，用于临时挂载存储设备|
|opt|第三方软件安装存放目录。|
|proc|进程及内核信息存放目录，不占用硬盘空间。|
|root|root用户的家目录|
|run|是一个临时文件系统，存储系统启动以来的信息。当系统重启时，这个目录下的文件应该被删掉或清除。|
|sbin|root用户使用的命令存放目录|
|srv|一些网络服务所需要的数据文件|
|sys|同proc目录，用于记录CPU与系统硬件的相关信息|
|tmp|程序运行时产生的临时文件存放目录|
|usr|系统存放程序的目录，类似于在windows下的文件夹programefiles|
|var|存放内容常变动的文件目录，如系统日志文件|
