
## 设备树
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/8c50fd6fbcf0641b5f58bab59af56894.png)

```txt
SPI3 0xFE640000 64KB
```

-->platform_device

**SPI3控制器设备树节点**：定义SPI3控制器的硬件配置参数，包括寄存器地址、中断、时钟和引脚配置
```c
// arch/arm64/boot/dts/rockchip/rk3066.dtsi
spi3: spi@fe640000 {
    compatible      = "rockchip,rk3066-spi";      // 兼容性字符串
    reg             = <0x0 0xfe640000 0x0 0x1000>; // 寄存器地址和大小
    interrupts      = <GIC_SPI 103 IRQ_TYPE_LEVEL_HIGH>; // 中断配置
    #address-cells  = <1>;                        // 地址单元格数量
    #size-cells     = <0>;                        // 大小单元格数量
    clocks          = <&cru CLK_SPI3>, <&cru PCLK_SPI3>; // 时钟源配置
    clock-names     = "spiclk", "apb_pclk";       // 时钟名称
    dmas            = <&dmac0 26>, <&dmac0 27>;    // DMA通道配置
    dma-names       = "tx", "rx";                 // DMA通道名称
    pinctrl-names   = "default", "high_speed";    // 引脚控制配置名称
    pinctrl-0       = <&spi3m0_cs0 &spi3m0_cs1 &spi3m0_pins>; // 默认引脚配置
    pinctrl-1       = <&spi3m0_cs0 &spi3m0_cs1 &spi3m0_pins_hs>; // 高速引脚配置
    status          = "disabled";                 // 设备状态
};
```

**设备树属性说明**：
- `compatible`：设备兼容性字符串，用于内核匹配相应的SPI控制器驱动程序
- `reg`：SPI控制器的物理寄存器基地址(0xfe640000)和地址空间大小(0x1000)
- `interrupts`：中断配置，使用GIC SPI中断103号，高电平触发方式
- `#address-cells/#size-cells`：定义子设备地址和大小的描述格式，用于SPI从设备节点
- `clocks`：引用的时钟源，包括SPI功能时钟和APB总线时钟
- `clock-names`：时钟的逻辑名称，驱动程序通过这些名称获取对应时钟
- `dmas/dma-names`：DMA传输配置，分别指定发送和接收使用的DMA通道
- `pinctrl-x`：引脚复用配置，包括片选信号、数据信号和时钟信号的GPIO配置
- `status`：设备初始状态，"disabled"表示默认禁用，需要在板级DTS中启用

其中需要注意pinctrl信息是否是我们需要的。


## 控制器驱动

/kernel/drivers/spi$ vim spi-rockchip.c

-->platform_driver

### 平台的注册
**Rockchip SPI设备树匹配表**：定义Rockchip系列芯片SPI控制器的设备树兼容性字符串匹配表
```c
// drivers/spi/spi-rockchip.c
static const struct of_device_id rockchip_spi_dt_match[] = {
    { .compatible = "rockchip,px30-spi",   },        // PX30芯片SPI控制器
    { .compatible = "rockchip,rv1108-spi", },        // RV1108芯片SPI控制器
    { .compatible = "rockchip,rv1126-spi", },        // RV1126芯片SPI控制器
    { .compatible = "rockchip,rk3036-spi", },        // RK3036芯片SPI控制器
    { .compatible = "rockchip,rk3066-spi", },        // RK3066芯片SPI控制器
    { .compatible = "rockchip,rk3188-spi", },        // RK3188芯片SPI控制器
    { .compatible = "rockchip,rk3228-spi", },        // RK3228芯片SPI控制器
    { .compatible = "rockchip,rk3288-spi", },        // RK3288芯片SPI控制器
    { .compatible = "rockchip,rk3368-spi", },        // RK3368芯片SPI控制器
    { .compatible = "rockchip,rk3399-spi", },        // RK3399芯片SPI控制器
    { },                                             // 数组结束标志
};
MODULE_DEVICE_TABLE(of, rockchip_spi_dt_match);      // 导出设备树匹配表
```

**Rockchip SPI平台驱动结构体**：定义Rockchip SPI控制器的平台驱动程序结构
```c
// drivers/spi/spi-rockchip.c
static struct platform_driver rockchip_spi_driver = {
    .driver = {
        .name            = DRIVER_NAME,                          // 驱动程序名称
        .pm              = &rockchip_spi_pm,                     // 电源管理操作
        .of_match_table  = of_match_ptr(rockchip_spi_dt_match),  // 设备树匹配表
    },
    .probe               = rockchip_spi_probe,                   // 设备探测函数
    .remove              = rockchip_spi_remove,                  // 设备移除函数
};
module_platform_driver(rockchip_spi_driver);                    // 注册平台驱动
```

**匹配表说明**：
- `px30/rv1108/rv1126`：Rockchip低功耗系列芯片的SPI控制器兼容性标识
- `rk3036/rk3066/rk3188`：Rockchip早期ARM Cortex-A系列芯片的SPI控制器
- `rk3228/rk3288`：Rockchip中端应用处理器系列的SPI控制器
- `rk3368/rk3399`：Rockchip高端64位应用处理器的SPI控制器
- `MODULE_DEVICE_TABLE(of, ...)`：向内核注册设备树匹配表，支持设备树自动匹配和模块加载

**驱动结构说明**：
- `DRIVER_NAME`：驱动程序的标识名称，通常为"rockchip-spi"
- `rockchip_spi_pm`：电源管理操作集，包括suspend/resume等回调函数
- `rockchip_spi_probe`：当匹配的设备被发现时调用的初始化函数
- `rockchip_spi_remove`：设备移除或驱动卸载时的清理函数
- `module_platform_driver`：简化的平台驱动注册宏，自动处理模块加载和卸载

### probe函数

1. 获取设备树资源
    
2. 向上层提供设备的操作函数接口（数据传输接口：transfer_one实现）
    
3. 向内核添加设备（顺便把挂在控制器上面的spidev进行注册）
    

瑞芯微的私有结构体，做记录用，后续数据传输使用：

  

```C
struct rockchip_spi {
        struct device *dev;

        struct clk *spiclk;
        struct clk *apb_pclk;

        void __iomem *regs;
        dma_addr_t dma_addr_rx;
        dma_addr_t dma_addr_tx;

        const void *tx;
        void *rx;
        unsigned int tx_left;
        unsigned int rx_left;

        atomic_t state;

        /*depth of the FIFO buffer */
        u32 fifo_len;
        /* frequency of spiclk */
        u32 freq;

        u8 n_bytes;
        u8 rsd;

        bool cs_asserted[ROCKCHIP_SPI_MAX_CS_NUM];

        struct pinctrl_state *high_speed_state;
        bool slave_abort;
        bool gpio_requested;
        bool cs_inactive; /* spi slave tansmition stop when cs inactive */
        struct spi_transfer *xfer; /* Store xfer temporarily */
        spinlock_t lock; /* prevent I/O concurrent access */
};
```

```C fold
static int rockchip_spi_probe(struct platform_device *pdev)
{
        int ret;
        struct rockchip_spi *rs;
        struct spi_controller *ctlr;
        struct resource *mem;
        struct device_node *np = pdev->dev.of_node;
        u32 rsd_nsecs;
        bool slave_mode;
        struct pinctrl *pinctrl = NULL;
        // 判断当前控制的模式
        slave_mode = of_property_read_bool(np, "spi-slave");


        if (slave_mode)
                ctlr = spi_alloc_slave(&pdev->dev,
                                sizeof(struct rockchip_spi));
        else
                ctlr = spi_alloc_master(&pdev->dev,
                                sizeof(struct rockchip_spi));

        //在platform_device 记录 master
        platform_set_drvdata(pdev, ctlr);

        //获取master data
        rs = spi_controller_get_devdata(ctlr);
        ctlr->slave = slave_mode;

        //对寄存器的映射
        /* Get basic io resource and map it */
        mem = platform_get_resource(pdev, IORESOURCE_MEM, 0);
        rs->regs = devm_ioremap_resource(&pdev->dev, mem);
        if (IS_ERR(rs->regs)) {
                ret =  PTR_ERR(rs->regs);
                goto err_put_ctlr;
        }

        //自旋锁的初始胡
        spin_lock_init(&rs->lock);
        //时钟的使能
        rs->apb_pclk = devm_clk_get(&pdev->dev, "apb_pclk");
        if (IS_ERR(rs->apb_pclk)) {
                dev_err(&pdev->dev, "Failed to get apb_pclk\n");
                ret = PTR_ERR(rs->apb_pclk);
                goto err_put_ctlr;
        }

        rs->spiclk = devm_clk_get(&pdev->dev, "spiclk");
        if (IS_ERR(rs->spiclk)) {
                dev_err(&pdev->dev, "Failed to get spi_pclk\n");
                ret = PTR_ERR(rs->spiclk);
                goto err_put_ctlr;
        }

        ret = clk_prepare_enable(rs->apb_pclk);
        if (ret < 0) {
                dev_err(&pdev->dev, "Failed to enable apb_pclk\n");
                goto err_put_ctlr;
        }

        ret = clk_prepare_enable(rs->spiclk);
        if (ret < 0) {
                dev_err(&pdev->dev, "Failed to enable spi_clk\n");
                goto err_disable_apbclk;
        }

        spi_enable_chip(rs, false);

//中断的注册
        ret = platform_get_irq(pdev, 0);
        if (ret < 0)
                goto err_disable_spiclk;

        ret = devm_request_threaded_irq(&pdev->dev, ret, rockchip_spi_isr, NULL,
                        IRQF_ONESHOT, dev_name(&pdev->dev), ctlr);
        if (ret)
                goto err_disable_spiclk;

        rs->dev = &pdev->dev;
        rs->freq = clk_get_rate(rs->spiclk);
        rs->gpio_requested = false;

         if (!of_property_read_u32(pdev->dev.of_node, "rx-sample-delay-ns",
                                  &rsd_nsecs)) {
                /* rx sample delay is expressed in parent clock cycles (max 3) */
                u32 rsd = DIV_ROUND_CLOSEST(rsd_nsecs * (rs->freq >> 8),
                                1000000000 >> 8);
                if (!rsd) {
                        dev_warn(rs->dev, "%u Hz are too slow to express %u ns delay\n",
                                        rs->freq, rsd_nsecs);
                } else if (rsd > CR0_RSD_MAX) {
                        rsd = CR0_RSD_MAX;
                        dev_warn(rs->dev, "%u Hz are too fast to express %u ns delay, clamping at %u ns\n",
                                        rs->freq, rsd_nsecs,
                                        CR0_RSD_MAX * 1000000000U / rs->freq);
                }
                rs->rsd = rsd;
        }
        
        //休眠唤醒机制
        pm_runtime_set_active(&pdev->dev);
        pm_runtime_enable(&pdev->dev);

        //spi_control 结构体的填充
        ctlr->auto_runtime_pm = true;
        ctlr->bus_num = pdev->id;
        ctlr->mode_bits = SPI_CPOL | SPI_CPHA | SPI_LOOP | SPI_LSB_FIRST | SPI_CS_HIGH;
        if (slave_mode) {
                ctlr->mode_bits |= SPI_NO_CS;
                ctlr->slave_abort = rockchip_spi_slave_abort;
        } else {
                ctlr->flags = SPI_MASTER_GPIO_SS;
        }
        ctlr->num_chipselect = ROCKCHIP_SPI_MAX_CS_NUM;
        ctlr->dev.of_node = pdev->dev.of_node;
        ctlr->bits_per_word_mask = SPI_BPW_MASK(16) | SPI_BPW_MASK(8) | SPI_BPW_MASK(4);
        ctlr->min_speed_hz = rs->freq / BAUDR_SCKDV_MAX;
        ctlr->max_speed_hz = min(rs->freq / BAUDR_SCKDV_MIN, MAX_SCLK_OUT);

        ctlr->set_cs = rockchip_spi_set_cs;
        ctlr->setup = rockchip_spi_setup;
        ctlr->cleanup = rockchip_spi_cleanup;
        ctlr->transfer_one = rockchip_spi_transfer_one;
        ctlr->max_transfer_size = rockchip_spi_max_transfer_size;
        ctlr->handle_err = rockchip_spi_handle_err;

        ctlr->dma_tx = dma_request_chan(rs->dev, "tx");

        //获取GPIO第二状态，休眠时候会用到
        pinctrl = devm_pinctrl_get(&pdev->dev);
        if (!IS_ERR(pinctrl)) {
                rs->high_speed_state = pinctrl_lookup_state(pinctrl, "high_speed");
                if (IS_ERR_OR_NULL(rs->high_speed_state)) {
                        dev_warn(&pdev->dev, "no high_speed pinctrl state\n");
                        rs->high_speed_state = NULL;
                }
        }

        //控制器注册
        ret = devm_spi_register_controller(&pdev->dev, ctlr);
        if (ret < 0) {
                dev_err(&pdev->dev, "Failed to register controller\n");
                goto err_free_dma_rx;
        }

        return 0;
```

下图展示了Rockchip SPI控制器的初始化配置过程，包括设置电源管理、SPI模式位、传输速度限制以及各种回调函数的赋值
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/0bb106fb4677289a43ed6847d7e674a5.png)

下图展示了SPI驱动中获取pinctrl配置、查找高速引脚状态，并向内核注册SPI控制器的过程，包含了相应的错误处理逻辑
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/deca4190518e781d7547a72d77d29d40.png)



## 控制器驱动结构体

linux/spi/spi.h

最新的内核代码中使用 spi_controller （以前是 spi_master 结构）来描述一个SPI的控制器

```C++ fold
/**
 * struct spi_controller - interface to SPI master or slave controller
 * @dev: device interface to this driver
 * @list: link with the global spi_controller list
 * @bus_num: board-specific (and often SoC-specific) identifier for a
 *      given SPI controller.
 * @num_chipselect: chipselects are used to distinguish individual
 *      SPI slaves, and are numbered from zero to num_chipselects.
 *      each slave has a chipselect signal, but it's common that not
 *      every chipselect is connected to a slave.
 * @dma_alignment: SPI controller constraint on DMA buffers alignment.
 * @mode_bits: flags understood by this controller driver
 * @bits_per_word_mask: A mask indicating which values of bits_per_word are
 *      supported by the driver. Bit n indicates that a bits_per_word n+1 is
 *      supported. If set, the SPI core will reject any transfer with an
 *      unsupported bits_per_word. If not set, this value is simply ignored,
 *      and it's up to the individual driver to perform any validation.
 * @min_speed_hz: Lowest supported transfer speed
 * @max_speed_hz: Highest supported transfer speed
 * @flags: other constraints relevant to this driver
 * @slave: indicates that this is an SPI slave controller
 * @max_transfer_size: function that returns the max transfer size for
 *      a &spi_device; may be %NULL, so the default %SIZE_MAX will be used.
 * @max_message_size: function that returns the max message size for
 *      a &spi_device; may be %NULL, so the default %SIZE_MAX will be used.
 * @io_mutex: mutex for physical bus access
 * @bus_lock_spinlock: spinlock for SPI bus locking
 * @bus_lock_mutex: mutex for exclusion of multiple callers
 * @bus_lock_flag: indicates that the SPI bus is locked for exclusive use
 * @setup: updates the device mode and clocking records used by a
 *      device's SPI controller; protocol code may call this.  This
 *      must fail if an unrecognized or unsupported mode is requested.
 *      It's always safe to call this unless transfers are pending on
 *      the device whose settings are being modified.
 * @set_cs_timing: optional hook for SPI devices to request SPI master
 * controller for configuring specific CS setup time, hold time and inactive
 * delay interms of clock counts
 * @transfer: adds a message to the controller's transfer queue.
 * @cleanup: frees controller-specific state
 * @can_dma: determine whether this controller supports DMA
 * @queued: whether this controller is providing an internal message queue
 * @kworker: thread struct for message pump
 * @kworker_task: pointer to task for message pump kworker thread
 * @pump_messages: work struct for scheduling work to the message pump
 * @queue_lock: spinlock to syncronise access to message queue
 * @queue: message queue
 * @idling: the device is entering idle state
 * @cur_msg: the currently in-flight message
 * @cur_msg_prepared: spi_prepare_message was called for the currently
 *                    in-flight message
 * @cur_msg_mapped: message has been mapped for DMA
 * @xfer_completion: used by core transfer_one_message()
 * @busy: message pump is busy
 * @running: message pump is running
 * @rt: whether this queue is set to run as a realtime task
 * @auto_runtime_pm: the core should ensure a runtime PM reference is held
 *                   while the hardware is prepared, using the parent
 *                   device for the spidev
* @max_dma_len: Maximum length of a DMA transfer for the device.
 * @prepare_transfer_hardware: a message will soon arrive from the queue
 *      so the subsystem requests the driver to prepare the transfer hardware
 *      by issuing this call
 * @transfer_one_message: the subsystem calls the driver to transfer a single
 *      message while queuing transfers that arrive in the meantime. When the
 *      driver is finished with this message, it must call
 *      spi_finalize_current_message() so the subsystem can issue the next
 *      message
 * @unprepare_transfer_hardware: there are currently no more messages on the
 *      queue so the subsystem notifies the driver that it may relax the
 *      hardware by issuing this call
 *
 * @set_cs: set the logic level of the chip select line.  May be called
 *          from interrupt context.
 * @prepare_message: set up the controller to transfer a single message,
 *                   for example doing DMA mapping.  Called from threaded
 *                   context.
 * @transfer_one: transfer a single spi_transfer.
 *                  - return 0 if the transfer is finished,
 *                  - return 1 if the transfer is still in progress. When
 *                    the driver is finished with this transfer it must
 *                    call spi_finalize_current_transfer() so the subsystem
 *                    can issue the next transfer. Note: transfer_one and
 *                    transfer_one_message are mutually exclusive; when both
 *                    are set, the generic subsystem does not call your
 *                    transfer_one callback.
 * @handle_err: the subsystem calls the driver to handle an error that occurs
 *              in the generic implementation of transfer_one_message().
 * @mem_ops: optimized/dedicated operations for interactions with SPI memory.
 *           This field is optional and should only be implemented if the
 *           controller has native support for memory like operations.
 * @unprepare_message: undo any work done by prepare_message().
 * @slave_abort: abort the ongoing transfer request on an SPI slave controller
 * @cs_gpios: LEGACY: array of GPIO descs to use as chip select lines; one per
 *      CS number. Any individual value may be -ENOENT for CS lines that
 *      are not GPIOs (driven by the SPI controller itself). Use the cs_gpiods
 *      in new drivers.
 * @cs_gpiods: Array of GPIO descs to use as chip select lines; one per CS
 *      number. Any individual value may be NULL for CS lines that
 *      are not GPIOs (driven by the SPI controller itself).
 * @use_gpio_descriptors: Turns on the code in the SPI core to parse and grab
 *      GPIO descriptors rather than using global GPIO numbers grabbed by the
 *      driver. This will fill in @cs_gpiods and @cs_gpios should not be used,
 *      and SPI devices will have the cs_gpiod assigned rather than cs_gpio.
 * @statistics: statistics for the spi_controller
 * @dma_tx: DMA transmit channel
 * @dma_rx: DMA receive channel
 * @dummy_rx: dummy receive buffer for full-duplex devices
 * @dummy_tx: dummy transmit buffer for full-duplex devices
 * @fw_translate_cs: If the boot firmware uses different numbering scheme
 *      what Linux expects, this optional hook can be used to translate
 *      between the two.
 *
 * Each SPI controller can communicate with one or more @spi_device
 * children.  These make a small bus, sharing MOSI, MISO and SCK signals
 * but not chip select signals.  Each device may be configured to use a
 * different clock rate, since those shared signals are ignored unless
 * the chip is selected.
 *
 * The driver for an SPI controller manages access to those devices through
 * a queue of spi_message transactions, copying data between CPU memory and
 * an SPI slave device.  For each such message it queues, it calls the
 * message's completion function when the transaction completes.
 */
struct spi_controller {
        struct device   dev;

        struct list_head list;

        /* other than negative (== assign one dynamically), bus_num is fully
         * board-specific.  usually that simplifies to being SoC-specific.
         * example:  one SoC has three SPI controllers, numbered 0..2,
         * and one board's schematics might show it using SPI-2.  software
         * would normally use bus_num=2 for that controller.
         */
        s16                     bus_num;

        /* chipselects will be integral to many controllers; some others
         * might use board-specific GPIOs.
         */
        u16                     num_chipselect;

        /* some SPI controllers pose alignment requirements on DMAable
         * buffers; let protocol drivers know about these requirements.
         */
        u16                     dma_alignment;

        /* spi_device.mode flags understood by this controller driver */
        u32                     mode_bits;

        /* bitmask of supported bits_per_word for transfers */
        u32                     bits_per_word_mask;
#define SPI_BPW_MASK(bits) BIT((bits) - 1)
#define SPI_BPW_RANGE_MASK(min, max) GENMASK((max) - 1, (min) - 1)

        /* limits on transfer speed */
        u32                     min_speed_hz;
        u32                     max_speed_hz;

        /* other constraints relevant to this driver */
        u16                     flags;
#define SPI_CONTROLLER_HALF_DUPLEX      BIT(0)  /* can't do full duplex */
#define SPI_CONTROLLER_NO_RX            BIT(1)  /* can't do buffer read */
#define SPI_CONTROLLER_NO_TX            BIT(2)  /* can't do buffer write */
#define SPI_CONTROLLER_MUST_RX          BIT(3)  /* requires rx */
#define SPI_CONTROLLER_MUST_TX          BIT(4)  /* requires tx */

#define SPI_MASTER_GPIO_SS              BIT(5)  /* GPIO CS must select slave */

        /* flag indicating this is an SPI slave controller */
        bool                    slave;
 /*
         * on some hardware transfer / message size may be constrained
         * the limit may depend on device transfer settings
         */
        size_t (*max_transfer_size)(struct spi_device *spi);
        size_t (*max_message_size)(struct spi_device *spi);

        /* I/O mutex */
        struct mutex            io_mutex;

        /* lock and mutex for SPI bus locking */
        spinlock_t              bus_lock_spinlock;
        struct mutex            bus_lock_mutex;

        /* flag indicating that the SPI bus is locked for exclusive use */
        bool                    bus_lock_flag;

        /* Setup mode and clock, etc (spi driver may call many times).
         *
         * IMPORTANT:  this may be called when transfers to another
         * device are active.  DO NOT UPDATE SHARED REGISTERS in ways
         * which could break those transfers.
         */
        int                     (*setup)(struct spi_device *spi);

        /*
         * set_cs_timing() method is for SPI controllers that supports
         * configuring CS timing.
         *
         * This hook allows SPI client drivers to request SPI controllers
         * to configure specific CS timing through spi_set_cs_timing() after
         * spi_setup().
         */
        void (*set_cs_timing)(struct spi_device *spi, u8 setup_clk_cycles,
                              u8 hold_clk_cycles, u8 inactive_clk_cycles);

        /* bidirectional bulk transfers
         *
         * + The transfer() method may not sleep; its main role is
         *   just to add the message to the queue.
         * + For now there's no remove-from-queue operation, or
         *   any other request management
         * + To a given spi_device, message queueing is pure fifo
         *
         * + The controller's main job is to process its message queue,
         *   selecting a chip (for masters), then transferring data
         * + If there are multiple spi_device children, the i/o queue
         *   arbitration algorithm is unspecified (round robin, fifo,
         *   priority, reservations, preemption, etc)
         *
         * + Chipselect stays active during the entire message
         *   (unless modified by spi_transfer.cs_change != 0).
         * + The message transfers use clock and SPI mode parameters
         *   previously established by setup() for this device
         */
        int                     (*transfer)(struct spi_device *spi,
                                                struct spi_message *mesg);

        /* called on release() to free memory provided by spi_controller */
        void                    (*cleanup)(struct spi_device *spi);

        /*
         * Used to enable core support for DMA handling, if can_dma()
         * exists and returns true then the transfer will be mapped
         * prior to transfer_one() being called.  The driver should
         * not modify or store xfer and dma_tx and dma_rx must be set
         * while the device is prepared.
         */
        bool                    (*can_dma)(struct spi_controller *ctlr,
                                           struct spi_device *spi,
                                           struct spi_transfer *xfer);
        /*
         * These hooks are for drivers that want to use the generic
         * controller transfer queueing mechanism. If these are used, the
         * transfer() function above must NOT be specified by the driver.
         * Over time we expect SPI drivers to be phased over to this API.
         */
        bool                            queued;
        struct kthread_worker           kworker;
        struct task_struct              *kworker_task;
        struct kthread_work             pump_messages;
        spinlock_t                      queue_lock;
        struct list_head                queue;
        struct spi_message              *cur_msg;
        bool                            idling;
        bool                            busy;
        bool                            running;
        bool                            rt;
        bool                            auto_runtime_pm;
        bool                            cur_msg_prepared;
        bool                            cur_msg_mapped;
        struct completion               xfer_completion;
        size_t                          max_dma_len;

        int (*prepare_transfer_hardware)(struct spi_controller *ctlr);
        int (*transfer_one_message)(struct spi_controller *ctlr,
                                    struct spi_message *mesg);
        int (*unprepare_transfer_hardware)(struct spi_controller *ctlr);
        int (*prepare_message)(struct spi_controller *ctlr,
                               struct spi_message *message);
        int (*unprepare_message)(struct spi_controller *ctlr,
                                 struct spi_message *message);
        int (*slave_abort)(struct spi_controller *ctlr);

        /*
         * These hooks are for drivers that use a generic implementation
         * of transfer_one_message() provied by the core.
         */
        void (*set_cs)(struct spi_device *spi, bool enable);
        int (*transfer_one)(struct spi_controller *ctlr, struct spi_device *spi,
                            struct spi_transfer *transfer);
        void (*handle_err)(struct spi_controller *ctlr,
                           struct spi_message *message);

        /* Optimized handlers for SPI memory-like operations. */
        const struct spi_controller_mem_ops *mem_ops;

        /* gpio chip select */
        int                     *cs_gpios;
        struct gpio_desc        **cs_gpiods;
        bool                    use_gpio_descriptors;

        /* statistics */
        struct spi_statistics   statistics;

        /* DMA channels for use with core dmaengine helpers */
        struct dma_chan         *dma_tx;
        struct dma_chan         *dma_rx;

        /* dummy data for full duplex devices */
        void                    *dummy_rx;
        void                    *dummy_tx;

        int (*fw_translate_cs)(struct spi_controller *ctlr, unsigned cs);
};
```

为了兼容以前的代码：
```c
/* Compatibility later */
#define spi_master spi_controller
```

其中部分参数含义如下：

- dev：spi_controller是一个device，所以包含了一个device的实例，设备模型使用；把spi_controller看做是device的子类；
    
    - class：spi_master_class；与I2C适配器比较，I2C适配设置了bus属性，因此是挂载I2C总线下，而SPI控制器与之不同；
        
- list：用于构建双向链表，链接到spi_controller list链表中；
    
- bus_num：SPI控制器的编号，比如某SoC有3个SPI控制，那么这个结构描述的是第几个；
    
- num_chipselect：片选数量，决定该控制器下面挂接多少个SPI设备，从设备的片选号不能大于这个数量；
    
- mode_bits：SPI 控制器支持模式标志位，比如：
    
    - SPI_CPHA：支持时钟相位选择；
        
    - SPI_CPOL：支持时钟记性选择；
        
    - SPI_CS_HIGH：片选信号为高电平；
        
    - ... 更多可以查看spi_device小节内容；
        
- min_speed_hz/max_speed_hz：最大最小速率；
    
- slave：是否是 slave；
    
- setup：SPI控制器初始化函数指针，用来设置SPI控制器和工作方式、clock等；
    
- transfer：添加消息到队列的方法。这个函数不可睡眠，它的职责是安排发生的传送并且调用注册的回调函 complete()。这个不同的控制器要具体实现，传输数据最后都要调用这个函数；
    
- cleanup：在spidev_release函数中被调用，spidev_release被登记为spi dev的release函数；
    
- set_cs：函数指针，可以用来实现SPI控制器片选信号，比如设置片选、取消片选，这个函数一般由SoC厂家的SPI控制器驱动程序提供；如果指定了cs_giops、cs_gpio，该参数将无效；
    
- cs_gpiod：片选GPIO描述符指针数组，如果一个SPI控制器外接了多个SPI从设备，这里存放的就是每个从设备的片选引脚GPIO描述符；
    
- cs_gpio：片选GPIO编号数组，如果一个SPI控制器外接了多个SPI从设备，这里存放的就是每个从设备的片选引脚GPIO编号；
    
- queue：消息队列，用于连接挂载该控制器下的spi_message结构；控制器上可以同时被加入多个spi_message进行排队；
    
- kworker：工作线程，负责执行工作的线程；
    
- kworker_task：调用kthread_run为kworker创建并启动一个内核线程来处理工作，创建返回的task_struct结构体；
    
- pump_messages：指的就是具体的工作（work），通过kthread_queue_work可以将工作挂到工作线程；
    
- busy: message pump is busy：指定是当前SPI是否正在进行数据传输；
    
- cur_msg：当前正在处理的spi_message；
    
- idling：内核工作线程是空闲的，即kworker没有工作需要执行；
    
- running: message pump is running：指的是消息队列是否启用，在spi_start_queue函数会将这个参数设置为true；不出意外的话，注册完SPI中断控制器后，这个参数时钟都是true；
    
- transfer、transfer_one、transfer_one_message：用于SPI数据传输；其区别在于：
    
    - transfer：添加一个message到SPI控制器传输消息队列；如果未指定，由SPI核心默认初始化为spi_queued_transfer；
        
    - transfer_one_message：传输一个spi_message，传输完成将会调用spi_finalize_current_message函数；由SPI核心默认初始化为spi_transfer_one_message；
        
    - transfer_one：传输一个spi_transfer，0：传输完成，1：传输进行中，传输完成需要调用spi_finalize_current_transfer函数；这个函数一般由SoC厂家的SPI控制器驱动程序提供；
        

## 控制器注册

### 分配spi_controller

```C++
static inline struct spi_controller *spi_alloc_master(struct device *host,
                                                      unsigned int size)
{
        return __spi_alloc_controller(host, size, false);
}
/**
 * __spi_alloc_controller - allocate an SPI master or slave controller
 * @dev: the controller, possibly using the platform_bus
 * @size: how much zeroed driver-private data to allocate; the pointer to this
 *      memory is in the driver_data field of the returned device,
 *      accessible with spi_controller_get_devdata().
 * @slave: flag indicating whether to allocate an SPI master (false) or SPI
 *      slave (true) controller
 * Context: can sleep
 *
 * This call is used only by SPI controller drivers, which are the
 * only ones directly touching chip registers.  It's how they allocate
 * an spi_controller structure, prior to calling spi_register_controller().
 *
 * This must be called from context that can sleep.
 *
 * The caller is responsible for assigning the bus number and initializing the
 * controller's methods before calling spi_register_controller(); and (after
 * errors adding the device) calling spi_controller_put() to prevent a memory
 * leak.
 *
 * Return: the SPI controller structure on success, else NULL.
 */
struct spi_controller *__spi_alloc_controller(struct device *dev,
                                              unsigned int size, bool slave)
{
        struct spi_controller   *ctlr;

        if (!dev)
                return NULL;

        ctlr = kzalloc(size + sizeof(*ctlr), GFP_KERNEL);  
        if (!ctlr)
                return NULL;

        device_initialize(&ctlr->dev);    // 初始化device
        ctlr->bus_num = -1;               // 设施编号
        ctlr->num_chipselect = 1;         // 设置片选数量
        ctlr->slave = slave;              // 不是从设备
        if (IS_ENABLED(CONFIG_SPI_SLAVE) && slave)
                ctlr->dev.class = &spi_slave_class;
        else
                ctlr->dev.class = &spi_master_class;  // 设置其class, 可以看到其并没有设备bus属性，也就是SPI控制器并不是挂在总线上的
        ctlr->dev.parent = dev;                       // 设置父节点，父节点一般是platform设备，因此其实挂载platform下的
        pm_suspend_ignore_children(&ctlr->dev, true);
        spi_controller_set_devdata(ctlr, &ctlr[1]);  // 设置私有数据ctrl->dev.driver_data = &ctrl[1]

        return ctlr;
}
```

SPI控制器和I2C适配器一样，一般都是基于platform模型的，这里传入的参数host便是xxx_spi_probe(struct platform_device *pdev) 中的 &pdev->dev 部分。

这个接口不仅仅是分配一个spi_controller的结构，它在 kzalloc的时候，其实是分配了一个 spi_controller + size 的结构，然后首地址为一个 spi_controller，并对其做了基本的初始化操作，最后调用：

```C++
spi_controller_set_devdata(ctlr, &ctlr[1]);
```

传入的参数是&ctlr[1] 也就是跳过了第一个ctlr的地址，将分配余下的size 大小的内存空间通过dev_set_drvdata的方式挂接到了 device->driver_data 下；如下图所示

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/f47624286ed2bdc02b77baa780d68868.png)


### 注册spi_controller

spi_register_controller
![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7f35c3ec9ee80839991f8a4a4794a618.png)


```C++ fold
/**
 * spi_register_controller - register SPI master or slave controller
 * @ctlr: initialized master, originally from spi_alloc_master() or
 *      spi_alloc_slave()
 * Context: can sleep
 *
 * SPI controllers connect to their drivers using some non-SPI bus,
 * such as the platform bus.  The final stage of probe() in that code
 * includes calling spi_register_controller() to hook up to this SPI bus glue.
 *
 * SPI controllers use board specific (often SoC specific) bus numbers,
 * and board-specific addressing for SPI devices combines those numbers
 * with chip select numbers.  Since SPI does not directly support dynamic
 * device identification, boards need configuration tables telling which
 * chip is at which address.
 *
 * This must be called from context that can sleep.  It returns zero on
 * success, else a negative error code (dropping the controller's refcount).
 * After a successful return, the caller is responsible for calling
 * spi_unregister_controller().
 *
 * Return: zero on success, else a negative error code.
 */
int spi_register_controller(struct spi_controller *ctlr)
{
        struct device           *dev = ctlr->dev.parent;   //获取其父，一般为platform设备dev
        struct boardinfo        *bi;
        int                     status;
        int                     id, first_dynamic;

        if (!dev)
                return -ENODEV;

        /*
         * Make sure all necessary hooks are implemented before registering
         * the SPI controller.
         */
        status = spi_controller_check_ops(ctlr);
        if (status)
                return status;

        if (ctlr->bus_num >= 0) {     // 如果指定了控制器编号
                /* devices with a fixed bus num must check-in with the num */
                mutex_lock(&board_lock);
                id = idr_alloc(&spi_master_idr, ctlr, ctlr->bus_num,  // 为spi_master_idr树新增一个节点，该节点在spi_master_idr树中有唯一的节点ID,即指定的ctrl->bus_num，并且将这个节点和ctrl关联，通过id就可以定位到这个ctrl
                        ctlr->bus_num + 1, GFP_KERNEL);
                mutex_unlock(&board_lock);
                if (WARN(id < 0, "couldn't get idr"))
                        return id == -ENOSPC ? -EBUSY : id;
                ctlr->bus_num = id;
        } else if (ctlr->dev.of_node) { // 设备树相关
                /* allocate dynamic bus number using Linux idr */
                id = of_alias_get_id(ctlr->dev.of_node, "spi");
                if (id >= 0) {
                        ctlr->bus_num = id;
                        mutex_lock(&board_lock);
                        id = idr_alloc(&spi_master_idr, ctlr, ctlr->bus_num,
                                       ctlr->bus_num + 1, GFP_KERNEL);
                        mutex_unlock(&board_lock);
                        if (WARN(id < 0, "couldn't get idr"))
                                return id == -ENOSPC ? -EBUSY : id;
                }
        }
        if (ctlr->bus_num < 0) { // 未指定控制器编号，这里自动分配一个
                first_dynamic = of_alias_get_highest_id("spi");  // 获取SPI控制器个数
                if (first_dynamic < 0)
                        first_dynamic = 0;
                else
                        first_dynamic++;

                mutex_lock(&board_lock);
                id = idr_alloc(&spi_master_idr, ctlr, first_dynamic, // 动态分配控制器编号，起始编号first_dynamic
                               0, GFP_KERNEL);
                mutex_unlock(&board_lock);
                if (WARN(id < 0, "couldn't get idr"))
                        return id;
                ctlr->bus_num = id;
        }
        INIT_LIST_HEAD(&ctlr->queue);  // 初始化双向链表
        spin_lock_init(&ctlr->queue_lock);  // 初始化自旋锁
        spin_lock_init(&ctlr->bus_lock_spinlock); // 初始化自旋锁
        mutex_init(&ctlr->bus_lock_mutex);  // 初始化互斥锁
        mutex_init(&ctlr->io_mutex);        // 初始化互斥锁  
        ctlr->bus_lock_flag = 0;
        init_completion(&ctlr->xfer_completion);
        if (!ctlr->max_dma_len)  // 未指定dma传输最大长度
                ctlr->max_dma_len = INT_MAX;

        /* register the device, then userspace will see it.
         * registration fails if the bus ID is in use.
         */
        dev_set_name(&ctlr->dev, "spi%u", ctlr->bus_num); // 设置SPI控制器设备名称为spi%d

        if (!spi_controller_is_slave(ctlr)) {   // 非从设备
                if (ctlr->use_gpio_descriptors) {
                        status = spi_get_gpio_descs(ctlr);
                        if (status)
                                return status;
                        /*
                         * A controller using GPIO descriptors always
                         * supports SPI_CS_HIGH if need be.
                         */
                        ctlr->mode_bits |= SPI_CS_HIGH;
                } else {
                        /* Legacy code path for GPIOs from DT */
                        status = of_spi_register_master(ctlr);
                        if (status)
                                return status;
                }
        }
        /*
         * Even if it's just one always-selected device, there must
         * be at least one chipselect.
         */
        if (!ctlr->num_chipselect)  // 片选数量为0 终止，也就是说没有设备挂载这个控制器上
                return -EINVAL;

        status = device_add(&ctlr->dev);  // 添加设备到系统 会在/sys/devices/platform/-spi.x/spi_master下创建spi%d目录
        if (status < 0) {
                /* free bus id */
                mutex_lock(&board_lock);
                idr_remove(&spi_master_idr, ctlr->bus_num);
                mutex_unlock(&board_lock);
                goto done;
        }
        dev_dbg(dev, "registered %s %s\n",
                        spi_controller_is_slave(ctlr) ? "slave" : "master",
                        dev_name(&ctlr->dev));

        /*
         * If we're using a queued driver, start the queue. Note that we don't
         * need the queueing logic if the driver is only supporting high-level
         * memory operations.
         */
        if (ctlr->transfer) {  
                dev_info(dev, "controller is unqueued, this is deprecated\n");
        } else if (ctlr->transfer_one || ctlr->transfer_one_message) {
                status = spi_controller_initialize_queue(ctlr);  // 0成功 其他值失败
                if (status) {  
                        device_del(&ctlr->dev);
                        /* free bus id */
                        mutex_lock(&board_lock);
                        idr_remove(&spi_master_idr, ctlr->bus_num);
                        mutex_unlock(&board_lock);
                        goto done;
                }
        }
        /* add statistics */
        spin_lock_init(&ctlr->statistics.lock);

        mutex_lock(&board_lock);   // 获取互斥锁
        list_add_tail(&ctlr->list, &spi_controller_list);      // 将当前控制前添加到全局链表spi_controll_list
        list_for_each_entry(bi, &board_list, list)             // 扫描board_list链表，注册SPI从设备
                spi_match_controller_to_boardinfo(ctlr, &bi->board_info);
        mutex_unlock(&board_lock);  // 释放互斥锁

        /* Register devices from the device tree and ACPI */
        of_register_spi_devices(ctlr);     // 通过设备树节点注册所有该控制器下的所有从设备
        acpi_register_spi_devices(ctlr);   // 通过mach文件注册所有该控制器下的所有从设备
done:
        return status;
}
```

这个函数中，其实是进行了一些必要的设备模型的操作和钩子函数的check，通过后，将SPI控制器设备模型添加系统，同时将SPI控制器挂接到全局的spi_controller_list链表上。具体如下：

- 检查传入的spi_controller结构中，挂接的ops是不是全的，也就是说SPI核心层需要的一些必要的钩子函数是不是已经指定好了，这里需要确定SPI 控制器的那个传输的函数transfer 、transfer_one、transfer_one_message 至少挂接一个才行，也就是说，芯片厂家必须实现至少一种控制实际 SoC 的传输 SPI 的方式，SPI核心层才能够正常运行；
    
- 一些链表，自旋锁、互斥锁、dma最大传输长度的初始化；
    
- 将设备模型添加到系统；
    
- 如果芯片厂家没有实现transfer钩子函数的话，那么至少使用了挂接了transfer_one或者transfer_one_message，这意味着将会使用SPI核心层的新的传输机制，这里初始化queue机制；
    
- 将 spi_controller 挂接到全局的spi_controller_list双向链表；
    
- 遍历borad_list链表，调用spi_match_controller_to_boardinfo注册SPI从设备；
    
- 通过设备树节点、mach文件盖住该控制器下的所有SPI从设备；
    

### spi_match_controller_to_boardinfo

spi_match_controller_to_boardinfo函数定义如下：
```c
static void spi_match_controller_to_boardinfo(struct spi_controller *ctrl,
                                             struct spi_board_info *bi)
{
    struct spi_device *dev; // 声明SPI设备指针

    // 检查控制器和板信息的总线号是否匹配
    if (ctrl->bus_num != bi->bus_num)
        return; // 不匹配则直接返回

    // 为控制器和板信息创建新的SPI设备
    dev = spi_new_device(ctrl, bi);
    
    // 检查设备创建是否成功
    if (!dev)
        // 设备创建失败，输出错误信息
        dev_err(ctrl->dev.parent, "can't create new device for %s\n",
                bi->modalias);
}
```

这个函数的主要功能是：

- 匹配SPI控制器和板级信息
- 验证总线号是否一致
- 创建新的SPI设备
- 处理设备创建失败的错误情况

剩下部分详见

### spi_controller_initialize_queue

在注册SPI控制器方法中调用了spi_controller_initialize_queue
```c
static int spi_controller_initialize_queue(struct spi_controller *ctrl)
{
    int ret; // 返回值变量

    // 设置传输函数为队列传输模式
    ctrl->transfer = spi_queued_transfer;
    
    // 如果没有设置单消息传输函数，使用默认的
    if (!ctrl->transfer_one_message)
        ctrl->transfer_one_message = spi_transfer_one_message;

    /* Initialize and start queue */
    // 初始化队列
    ret = spi_init_queue(ctrl);
    if (ret) {
        // 队列初始化失败，输出错误信息
        dev_err(&ctrl->dev, "problem initializing queue\n");
        goto err_init_queue; // 跳转到错误处理
    }
    
    // 标记队列已初始化
    ctrl->queued = true;
    
    // 启动队列
    ret = spi_start_queue(ctrl);
    if (ret) {
        // 队列启动失败，输出错误信息
        dev_err(&ctrl->dev, "problem starting queue\n");
        goto err_start_queue; // 跳转到错误处理
    }

    return 0; // 成功返回0

err_start_queue:
    spi_destroy_queue(ctrl); // 销毁队列
err_init_queue:
    return ret; // 返回错误码
}
```

这个函数的主要功能是：

- 初始化SPI控制器的队列传输模式
- 设置传输函数指针
- 初始化并启动消息队列
- 包含完整的错误处理机制


该函数主要主要：

- 先将 transfer 赋值成为 spi_queued_transfer 调用（因为只有当 transfer 没有挂指针的时候，才会走进这个函数），如果没有指定 transfer_one_message，那么为他它上SPI 核心层的 spi_transfer_one_message；这样spi_controller->transfer 和 spi_controller->spi_transfer_one_message 已经赋值为SPI核心层层的函数钩子；
    
- 接着调用spi_init_queue和spi_start_queue函数；
    

#### spi_init_queue

```C++
static int spi_init_queue(struct spi_controller *ctlr)
{
        struct sched_param param = { .sched_priority = MAX_RT_PRIO - 1 };

        ctlr->running = false;
        ctlr->busy = false;

        kthread_init_worker(&ctlr->kworker);      // 初始化工作线程
        ctlr->kworker_task = kthread_run(kthread_worker_fn, &ctlr->kworker,   // 创建并启动一个内核线程来处理工作
                                         "%s", dev_name(&ctlr->dev));
        if (IS_ERR(ctlr->kworker_task)) {
                dev_err(&ctlr->dev, "failed to create message pump task\n");
                return PTR_ERR(ctlr->kworker_task);
        }
        kthread_init_work(&ctlr->pump_messages, spi_pump_messages);   // 初始化kthread_work，并设置work执行函数

        /*
         * Controller config will indicate if this controller should run the
         * message pump with high (realtime) priority to reduce the transfer
         * latency on the bus by minimising the delay between a transfer
         * request and the scheduling of the message pump thread. Without this
         * setting the message pump thread will remain at default priority.
         */
        if (ctlr->rt) {
                dev_info(&ctlr->dev,
                        "will run message pump with realtime priority\n");
                sched_setscheduler(ctlr->kworker_task, SCHED_FIFO, &param);
        }

        return 0;
}
```

在spi_init_queue时，会初始化&ctlr->pump_messages为spi_pump_messages，其内部调用了__spi_pump_message函数。

```C++
/**
 * spi_pump_messages - kthread work function which processes spi message queue
 * @work: pointer to kthread work struct contained in the controller struct
 */
static void spi_pump_messages(struct kthread_work *work)
{
        struct spi_controller *ctlr =
                container_of(work, struct spi_controller, pump_messages);

        __spi_pump_messages(ctlr, true);
}
```

#### spi_start_queue

spi_start_queue函数调用比较简单，直接running状态为 true，并设置当前cur_msg 为NULL，并将pump_messages挂载工作线程kthread_worker 上，这个 work执行函就是上面设置的 spi_pump_messages。

```C++
static int spi_start_queue(struct spi_controller *ctlr)
{
        unsigned long flags;

        spin_lock_irqsave(&ctlr->queue_lock, flags);  // 获取自旋锁，并关中断（防止中断发生，也试图获取自旋锁，造成死锁问题）

        if (ctlr->running || ctlr->busy) {
                spin_unlock_irqrestore(&ctlr->queue_lock, flags);
                return -EBUSY;
        }

        ctlr->running = true;
        ctlr->cur_msg = NULL;
        spin_unlock_irqrestore(&ctlr->queue_lock, flags);    // 释放自旋锁，并开中断

        kthread_queue_work(&ctlr->kworker, &ctlr->pump_messages);  // 将work加入到worker

        return 0;
}
```

#### spi_pump_messages

```c
/**
 * spi_pump_messages - kthread work function which processes spi message queue
 * @work: pointer to kthread work struct contained in the controller struct
 */
static void spi_pump_messages(struct kthread_work *work)
{
    // 通过container_of宏从work结构体获取包含它的spi_controller结构体指针
    struct spi_controller *ctrl =
        container_of(work, struct spi_controller, pump_messages);

    // 调用实际的消息处理函数，参数true表示启用某种处理模式
    __spi_pump_messages(ctrl, true);
}
```

这个函数的主要功能是：

- 作为内核线程的工作函数，用于处理SPI消息队列
- 从工作结构体中提取SPI控制器指针
- 调用底层的消息处理函数来实际处理队列中的消息

```C++
static void __spi_pump_messages(struct spi_controller *ctlr, bool in_kthread)
{
        unsigned long flags;
        bool was_busy = false;
        int ret;
        ...
        // spi_message结构体，spi_controller->queue链表链接着一系列spi_message
        // 获取链表中的第一个spi_message结构体，每次添加时都是添加到链表末尾
        ctlr->cur_msg =
                list_first_entry(&ctlr->queue, struct spi_message, queue);
        // 将第一个spi_message从上面的链表中删除
        list_del_init(&ctlr->cur_msg->queue);
        ...
        // 函数指针，前面已经初始化好
        if (ctlr->prepare_message) {
                ret = ctlr->prepare_message(ctlr, ctlr->cur_msg);
                ...
                ctlr->cur_msg_prepared = true;
        }
        ...
        // 重要，发送，就是函数 spi_transfer_one_message
        ret = ctlr->transfer_one_message(ctlr, ctlr->cur_msg);
        ...
}
```

```C++
static int spi_transfer_one_message(struct spi_controller *ctlr,
                                    struct spi_message *msg)
{
        struct spi_transfer *xfer;
        bool keep_cs = false;
        int ret = 0;
        unsigned long long ms = 1;
        struct spi_statistics *statm = &ctlr->statistics;
        struct spi_statistics *stats = &msg->spi->statistics;
        ...
        list_for_each_entry(xfer, &msg->transfers, transfer_list) {
                ...
                if (xfer->tx_buf || xfer->rx_buf) {
                        reinit_completion(&ctlr->xfer_completion);
                        // 其实就是，真正负责spi消息的发送
                        ret = ctlr->transfer_one(ctlr, msg->spi, xfer);
                        ...
                        if (ret > 0) {
                                ret = 0;
                                ms = 8LL * 1000LL * xfer->len;
                                do_div(ms, xfer->speed_hz);
                                ms += ms + 200; /* some tolerance */

                                if (ms > UINT_MAX)
                                        ms = UINT_MAX;

                                ms = wait_for_completion_timeout(&ctlr->xfer_completion,
                                                                 msecs_to_jiffies(ms));
                        }
                
                    }
           ...
           // 详见下
           spi_finalize_current_message(ctlr);
           ...
}
```

```C++
void spi_finalize_current_message(struct spi_controller *ctlr)
{
        struct spi_message *mesg;
        unsigned long flags;
        int ret;
        
        ...
        ctlr->cur_msg = NULL;
        ctlr->cur_msg_prepared = false;
        // 为内核worker添加一个新的处理工作，即spi_pump_message
        kthread_queue_work(&ctlr->kworker, &ctlr->pump_messages);
        ...
        mesg->state = NULL;
        // 不为空则唤醒之前阻塞的线程
        if (mesg->complete)
                mesg->complete(mesg->context);
}
```

#### spi_transfer_one_message

```C++
static int spi_transfer_one_message(struct spi_controller *ctlr,
                                    struct spi_message *msg)
{
        struct spi_transfer *xfer;
        bool keep_cs = false;
        int ret = 0;
        unsigned long long ms = 1;
        struct spi_statistics *statm = &ctlr->statistics;
        struct spi_statistics *stats = &msg->spi->statistics;
 
        spi_set_cs(msg->spi, true);
 
        SPI_STATISTICS_INCREMENT_FIELD(statm, messages);
        SPI_STATISTICS_INCREMENT_FIELD(stats, messages);
 
        list_for_each_entry(xfer, &msg->transfers, transfer_list) {
                trace_spi_transfer_start(msg, xfer);
 
                spi_statistics_add_transfer_stats(statm, xfer, ctlr);
                spi_statistics_add_transfer_stats(stats, xfer, ctlr);
 
                if (xfer->tx_buf || xfer->rx_buf) {
                        reinit_completion(&ctlr->xfer_completion);
 
                        ret = ctlr->transfer_one(ctlr, msg->spi, xfer);
                        if (ret < 0) {
                                SPI_STATISTICS_INCREMENT_FIELD(statm,
                                                               errors);
                                SPI_STATISTICS_INCREMENT_FIELD(stats,
                                                               errors);
                                dev_err(&msg->spi->dev,
                                        "SPI transfer failed: %d\n", ret);
                                goto out;
                        }
 
                        if (ret > 0) {
                                ret = 0;
                                ms = 8LL * 1000LL * xfer->len;
                                do_div(ms, xfer->speed_hz);
                                ms += ms + 200; /* some tolerance */
 
                                if (ms > UINT_MAX)
                                        ms = UINT_MAX;
 
                                ms = wait_for_completion_timeout(&ctlr->xfer_completion,
                                                                 msecs_to_jiffies(ms));
                        }
 
                        if (ms == 0) {
                                SPI_STATISTICS_INCREMENT_FIELD(statm,
                                                               timedout);
                                SPI_STATISTICS_INCREMENT_FIELD(stats,
                                                               timedout);
                                dev_err(&msg->spi->dev,
                                        "SPI transfer timed out\n");
                                msg->status = -ETIMEDOUT;
                        }
                } else {
                        if (xfer->len)
                                dev_err(&msg->spi->dev,
                                        "Bufferless transfer has length %u\n",
                                        xfer->len);
                }
 
                trace_spi_transfer_stop(msg, xfer);
 
                if (msg->status != -EINPROGRESS)
                        goto out;
 
                if (xfer->delay_usecs) {
                        u16 us = xfer->delay_usecs;
 
                        if (us <= 10)
                                udelay(us);
                        else
                                usleep_range(us, us + DIV_ROUND_UP(us, 10));
                }
 
                if (xfer->cs_change) {
                        if (list_is_last(&xfer->transfer_list,
                                         &msg->transfers)) {
                                keep_cs = true;
                        } else {
                                spi_set_cs(msg->spi, false);
                                udelay(10);
                                spi_set_cs(msg->spi, true);
                        }
                }
 
                msg->actual_length += xfer->len;
        }
 
out:
        if (ret != 0 || !keep_cs)
                spi_set_cs(msg->spi, false);
 
        if (msg->status == -EINPROGRESS)
                msg->status = ret;
 
        if (msg->status && ctlr->handle_err)
                ctlr->handle_err(ctlr, msg);
 
        spi_res_release(ctlr, msg);
 
        spi_finalize_current_message(ctlr);
 
        return ret;
}
```

这里我们就要发起传输了：

1. 首先设置了片选 cs，具体调用到了芯片厂家底层对接的 set_cs
    
2. 从 spi_message 中的 transfer_list 逐个取出 transfer 调用底层对接的 transfer_one
    
3. 根据是否配置了需要的延时 delay_usecs，来确定是否在下一次传输之前进行必要的 udelay
    
4. 根据是否配置了 cs_change，来判断是否需要变动片选信号 cs
    
5. 累加实际传输的数据 actual_length
    
6. 调用 spi_finalize_current_message 告知完成传输