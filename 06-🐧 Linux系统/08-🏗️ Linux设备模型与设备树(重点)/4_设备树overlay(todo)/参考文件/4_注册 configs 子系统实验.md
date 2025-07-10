注册一个 configfs 子系统

```C
#include <linux/module.h>
#include <linux/init.h>
#include <linux/slab.h>
#include <linux/configfs.h>

//定义名为"myconfig_item_type"的配置项类型结构体
static const struct config_item_type myconfig_item_type ={
  .ct_owner = THIS_MODULE,
  .ct_item_ops = NULL,
  .ct_group_ops = NULL,
  .ct_attrs = NULL,

};
//定义一个configfs_subsystem结构体实例"myconfigfs_subsystem"
static struct configfs_subsystem myconfigfs_subsystem ={
  .su_group = {
    .cg_item = {
      .ci_namebuf = "myconfigfs",
      .ci_type = &myconfig_item_type,
    },
  },

};

//模块的初始化函数
static int myconfigfs_init(void)
{
  //初始化配置组
  config_group_init(&myconfigfs_subsystem.su_group);
  //注册子系统
  configfs_register_subsystem(&myconfigfs_subsystem);
  return 0;
}
// 模块退出函数
static void myconfigfs_exit(void)
{
  configfs_unregister_subsystem(&myconfigfs_subsystem);

}

module_init(myconfigfs_init); // 指定模块的初始化函数
module_exit(myconfigfs_exit); // 指定模块的退出函数

MODULE_LICENSE("GPL");   // 模块使用的许可证
```