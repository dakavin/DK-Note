在学习I2C驱动的时候，针对 I2C 设备寄存器的操作都是通过相关的 API 函数进行操作的。这样 Linux 内核中就会充斥着大量的重复、冗余代码，但是这些本质上都是对寄存器的操作，所以为了方便内核开发人员统一访问 I2C 设备的时候，为此引入了 Regmap 子系统，本章我们就来学习一下如何使用 Regmap API 函数来读写 I2C 设备寄存器。

## 什么是Regmap

Linux 下大部分设备的驱动开发都是操作其内部寄存器，比如 I2C/SPI 设备的本质都是一样的，通过 I2C/SPI 接口读写芯片内部寄存器。芯片内部寄存器也是同样的道理，比如RK3568 的 PWM、TIM 等外设初始化，最终都是要落到寄存器的设置上。

Linux 下使用 i2c_transfer 来读写 I2C 设备中的寄存器，SPI 接口的话使用 spi_write/spi_read等。I2C/SPI 芯片又非常的多，因此 Linux 内核里面就会充斥了大量的 i2c_transfer 这类的冗余代码，同时代码的复用性也会降低。

基于代码复用的原则，Linux 内核引入了 regmap 模型，regmap 将寄存器访问的共同逻辑抽象出来，驱动开发人员不需要再去纠结使用 SPI 或者 I2C 接口 API 函数，统一使用 regmap API函数。这样的好处就是统一使用 regmap，降低了代码冗余，提高了驱动的可以移植性。

regmap模型的重点在于：

通过 regmap 模型提供的统一接口函数来访问器件的寄存器，SOC 内部的寄存器也可以使用 regmap 接口函数来访问。

regmap 是 Linux 内核为了减少慢速 I/O 在驱动上的冗余开销，提供了一种通用的接口来操作硬件寄存器。另外，regmap 在驱动和硬件之间添加了cache，降低了低速 I/O 的操作次数，提高了访问效率，缺点是实时性会降低。

## Regmap的适用场景

什么情况下会使用 regmap?

1、硬件寄存器操作，比如选用通过 I2C/SPI 接口来读写设备的内部寄存器，或者需要读写 SOC 内部的硬件寄存器。

2、提高代码复用性和驱动一致性，简化驱动开发过程。

3、减少底层 I/O 操作次数，提高访问效率。

## Regmap驱动框架

regmap 驱动框架如下图所示：

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/a817780392d874e7adfbc1ac9f164f0b.png)


regmap 框架分为三层：

①、底层物理总线：regmap 就是对不同的物理总线进行封装，目前 regmap 支持的物理总线有 i2c、i3c、spi、mmio、sccb、sdw、slimbus、irq、spmi 和 w1。

②、regmap 核心层，用于实现 regmap，我们不用关心具体实现。

③、regmapAPI 抽象层，regmap 向驱动编写人员提供的 API 接口，驱动编写人员使用这些API 接口来操作具体的芯片设备，也是驱动编写人员重点要掌握的。

## Regmap相关结构体

### regmap 结 构 体

Linux 内 核 将 regmap 框 架 抽 象 为 regmap 结 构 体 ， 这 个 结 构 体 定 义 在 文 件 drivers/base/regmap/internal.h 中

```C
struct regmap {
    union {
        struct mutex mutex;
        struct {
            spinlock_t spinlock;
            unsigned long spinlock_flags;
        };
    };
    regmap_lock lock;
    regmap_unlock unlock;
    void *lock_arg; /* This is passed to lock/unlock functions */
    gfp_t alloc_flags;

    struct device *dev; /* Device we do I/O on */
    void *work_buf;     /* Scratch buffer used to format I/O */
    struct regmap_format format;  /* Buffer format */
    const struct regmap_bus *bus;
    void *bus_context;
    const char *name;

    bool async;
    spinlock_t async_lock;
    wait_queue_head_t async_waitq;
    struct list_head async_list;
    struct list_head async_free;
    int async_ret;

#ifdef CONFIG_DEBUG_FS
    bool debugfs_disable;
    struct dentry *debugfs;
    const char *debugfs_name;

    unsigned int debugfs_reg_len;
    unsigned int debugfs_val_len;
    unsigned int debugfs_tot_len;

    struct list_head debugfs_off_cache;
    struct mutex cache_lock;
#endif

    unsigned int max_register;
    bool (*writeable_reg)(struct device *dev, unsigned int reg);
    bool (*readable_reg)(struct device *dev, unsigned int reg);
    bool (*volatile_reg)(struct device *dev, unsigned int reg);
    bool (*precious_reg)(struct device *dev, unsigned int reg);
    bool (*writeable_noinc_reg)(struct device *dev, unsigned int reg);
    bool (*readable_noinc_reg)(struct device *dev, unsigned int reg);
    const struct regmap_access_table *wr_table;
    const struct regmap_access_table *rd_table;
    const struct regmap_access_table *volatile_table;
    const struct regmap_access_table *precious_table;
    const struct regmap_access_table *wr_noinc_table;
    const struct regmap_access_table *rd_noinc_table;

    int (*reg_read)(void *context, unsigned int reg, unsigned int *val);
    int (*reg_write)(void *context, unsigned int reg, unsigned int val);
    int (*reg_update_bits)(void *context, unsigned int reg,
                   unsigned int mask, unsigned int val);

    bool defer_caching;

    unsigned long read_flag_mask;
    unsigned long write_flag_mask;

    /* number of bits to (left) shift the reg value when formatting*/
    int reg_shift;
    int reg_stride;
    int reg_stride_order;

    /* regcache specific members */
    const struct regcache_ops *cache_ops;
    enum regcache_type cache_type;

    /* number of bytes in reg_defaults_raw */
    unsigned int cache_size_raw;
    /* number of bytes per word in reg_defaults_raw */
    unsigned int cache_word_size;
    /* number of entries in reg_defaults */
    unsigned int num_reg_defaults;
    /* number of entries in reg_defaults_raw */
    unsigned int num_reg_defaults_raw;

    /* if set, only the cache is modified not the HW */
    bool cache_only;
    /* if set, only the HW is modified not the cache */
    bool cache_bypass;
    /* if set, remember to free reg_defaults_raw */
    bool cache_free;

    struct reg_default *reg_defaults;
    const void *reg_defaults_raw;
    void *cache;
    /* if set, the cache contains newer data than the HW */
    bool cache_dirty;
    /* if set, the HW registers are known to match map->reg_defaults */
    bool no_sync_defaults;

    struct reg_sequence *patch;
    int patch_regs;

    /* if set, converts bulk read to single read */
    bool use_single_read;
    /* if set, converts bulk write to single write */
    bool use_single_write;
    /* if set, the device supports multi write mode */
    bool can_multi_write;

    /* if set, raw reads/writes are limited to this size */
    size_t max_raw_read;
    size_t max_raw_write;

    struct rb_root range_tree;
    void *selector_work_buf;    /* Scratch buffer used for selector */

    struct hwspinlock *hwlock;

    /* if set, the regmap core can sleep */
    bool can_sleep;
};
```

### regmap_config 结构体

regmap_config 结构体就是用来初始化 regmap 的，这个结构体定义在include/linux/regmap.h 文件中，结构体内容如下：

```C
struct regmap_config {
    const char *name;

    int reg_bits;
    int reg_stride;
    int pad_bits;
    int val_bits;

    bool (*writeable_reg)(struct device *dev, unsigned int reg);
    bool (*readable_reg)(struct device *dev, unsigned int reg);
    bool (*volatile_reg)(struct device *dev, unsigned int reg);
    bool (*precious_reg)(struct device *dev, unsigned int reg);
    bool (*writeable_noinc_reg)(struct device *dev, unsigned int reg);
    bool (*readable_noinc_reg)(struct device *dev, unsigned int reg);

    bool disable_locking;
    regmap_lock lock;
    regmap_unlock unlock;
    void *lock_arg;

    int (*reg_read)(void *context, unsigned int reg, unsigned int *val);
    int (*reg_write)(void *context, unsigned int reg, unsigned int val);

    bool fast_io;

    unsigned int max_register;
    const struct regmap_access_table *wr_table;
    const struct regmap_access_table *rd_table;
    const struct regmap_access_table *volatile_table;
    const struct regmap_access_table *precious_table;
    const struct regmap_access_table *wr_noinc_table;
    const struct regmap_access_table *rd_noinc_table;
    const struct reg_default *reg_defaults;
    unsigned int num_reg_defaults;
    enum regcache_type cache_type;
    const void *reg_defaults_raw;
    unsigned int num_reg_defaults_raw;

    unsigned long read_flag_mask;
    unsigned long write_flag_mask;
    bool zero_flag_mask;

    bool use_single_read;
    bool use_single_write;
    bool can_multi_write;

    enum regmap_endian reg_format_endian;
    enum regmap_endian val_format_endian;

    const struct regmap_range_cfg *ranges;
    unsigned int num_ranges;

    bool use_hwlock;
    unsigned int hwlock_id;
    unsigned int hwlock_mode;

    bool can_sleep;
};
```

## Regmap操作函数

### Regmap 申请与初始化

regmap 支持多种物理总线，比如 I2C 和 SPI，我们需要根据所使用的接口来选择合适的 regmap 初始化函数。这里以I2C为例

Linux 内核提供了针对不同接口的 regmap 初始化函数，I2C接口初始化函数为 regmap_init_i2c，函数原型如下：

```C
#define devm_regmap_init_i2c(i2c, config)               \
    __regmap_lockdep_wrapper(__devm_regmap_init_i2c, #config,   \
                i2c, config)


struct regmap *__devm_regmap_init_i2c(struct i2c_client *i2c,
                      const struct regmap_config *config,
                      struct lock_class_key *lock_key,
                      const char *lock_name);
```

  

### regmap 设备访问 API 函数

不管是 I2C 还是 SPI 等接口，还是 SOC 内部的寄存器，对于寄存器的操作就两种：读和写。regmap 提供了最核心的两个读写操作：regmap_read 和regmap_write。这两个函数分别用来读/写寄存器.

regmap_read ：

```C
int regmap_read(struct regmap *map, unsigned int reg, unsigned int *val);
```

函数参数和返回值含义如下：

**map：**要操作的 regmap**。**

**reg：**要读的寄存器。

**val**：读到的寄存器值。

**返回值**：0，读取成功；其他值，读取失败。

regmap_write:

```C
int regmap_write(struct regmap *map, unsigned int reg, unsigned int val);
```

函数参数和返回值含义如下：

**map：**要操作的 regmap**。**

**reg：**要写的寄存器。

**val**：要写的寄存器值。

**返回值**：0，写成功；其他值，写失败

除此之外，还衍生了一些其他的API函数，具体可以参考文件include/linux/regmap.h

```C
int regmap_mmio_attach_clk(struct regmap *map, struct clk *clk);
void regmap_mmio_detach_clk(struct regmap *map);
void regmap_exit(struct regmap *map);
int regmap_reinit_cache(struct regmap *map,
            const struct regmap_config *config);
struct regmap *dev_get_regmap(struct device *dev, const char *name);
struct device *regmap_get_device(struct regmap *map);
int regmap_write(struct regmap *map, unsigned int reg, unsigned int val);
int regmap_write_async(struct regmap *map, unsigned int reg, unsigned int val);
int regmap_raw_write(struct regmap *map, unsigned int reg,
             const void *val, size_t val_len);
int regmap_noinc_write(struct regmap *map, unsigned int reg,
             const void *val, size_t val_len);
int regmap_bulk_write(struct regmap *map, unsigned int reg, const void *val,
            size_t val_count);
int regmap_multi_reg_write(struct regmap *map, const struct reg_sequence *regs,
            int num_regs);
int regmap_multi_reg_write_bypassed(struct regmap *map,
                    const struct reg_sequence *regs,
                    int num_regs);
int regmap_raw_write_async(struct regmap *map, unsigned int reg,
               const void *val, size_t val_len);
int regmap_read(struct regmap *map, unsigned int reg, unsigned int *val);
int regmap_raw_read(struct regmap *map, unsigned int reg,
            void *val, size_t val_len);
int regmap_noinc_read(struct regmap *map, unsigned int reg,
              void *val, size_t val_len);
int regmap_bulk_read(struct regmap *map, unsigned int reg, void *val,
             size_t val_count);
int regmap_update_bits_base(struct regmap *map, unsigned int reg,
                unsigned int mask, unsigned int val,
                bool *change, bool async, bool force);
```