
中断控制器通过IRQCHIP_DECLARE 宏注册到__irqchip_of_table
```c
// 内核源码/drivers/irqchip/irq-gic-v3.c
//初始化一个struct of_device_id的静态常量，并放置在__irqchip_of_table中
IRQCHIP_DECLARE(gic_v3, "arm,gic-v3", gicv3_of_init);  
```

系统开机初始化阶段,of_irq_init函数会去查找设备节点信息，根据__irqchip_of_table段中的”arm,gic-v3”，匹配到前面小节的设备树节点“gic”，获取设备信息。 最终会执行IRQCHIP_DECLARE声明的回调函数gicv3_of_init。

在内核源码include/linux/irqchip.h文件中定义了IRQCHIP_DECLARE宏:
```c
// 内核源码include/linux/irqchip.h

#define IRQCHIP_DECLARE(name, compat, fn) OF_DECLARE_2(irqchip, name, compat, fn)

#define OF_DECLARE_2(table, name, compat, fn) \
        _OF_DECLARE(table, name, compat, fn, of_init_fn_2)

#define _OF_DECLARE(table, name, compat, fn, fn_type)                       \
    static const struct of_device_id __of_table_##name              \
        __used __section(__##table##_of_table)                      \
        __aligned(__alignof__(struct of_device_id))         \
        = { .compatible = compat,                           \
            .data = (fn == (fn_type)NULL) ? fn : fn  }
```

该宏初始化了一个struct of_device_id静态常量，放到并放置在__irqchip_of_table段中

```c
// 内核源码/drivers/irqchip/irq-gic-v3.c

static int __init gicv3_of_init(struct device_node *node, struct device_node *parent)
{
    void __iomem *dist_base;
    struct redist_region *rdist_regs;
    u64 redist_stride;
    u32 nr_redist_regions;
    int err, i;

    dist_base = of_iomap(node, 0);           //映射 GICD 的寄存器地址空间
    if (!dist_base) {
        pr_err("%pOF: unable to map gic dist registers\n", node);
        return -ENXIO;
    }

    err = gic_validate_dist_version(dist_base);       //验证GIC硬件的版本是GICv3或者GICv4
    if (err) {
        pr_err("%pOF: no distributor detected, giving up\n", node);
        goto out_unmap_dist;
    }

    /*读取设备树节点redistributor-regions 的值,没有就是1*/
    if (of_property_read_u32(node, "#redistributor-regions", &nr_redist_regions))
        nr_redist_regions = 1;

    rdist_regs = kcalloc(nr_redist_regions, sizeof(*rdist_regs),
                GFP_KERNEL);
    if (!rdist_regs) {
        err = -ENOMEM;
        goto out_unmap_dist;
    }

    for (i = 0; i < nr_redist_regions; i++) { //为一个 GICR 域分配基地址
        struct resource res;
        int ret;

        ret = of_address_to_resource(node, 1 + i, &res);
        rdist_regs[i].redist_base = of_iomap(node, 1 + i);
        if (ret || !rdist_regs[i].redist_base) {
            pr_err("%pOF: couldn't map region %d\n", node, i);
            err = -ENODEV;
            goto out_unmap_rdist;
        }
        rdist_regs[i].phys_base = res.start;
    }

    /*通过 DTS 读取 redistributor-regions 的值,没有就是0*/
    if (of_property_read_u64(node, "redistributor-stride", &redist_stride))
        redist_stride = 0;

    /*GIC v3初始化的核心工作*/
    err = gic_init_bases(dist_base, rdist_regs, nr_redist_regions,
                redist_stride, &node->fwnode);
    if (err)
        goto out_unmap_rdist;

    gic_populate_ppi_partitions(node);   //设置PPI的亲和性

    if (static_branch_likely(&supports_deactivate_key))
        gic_of_setup_kvm_info(node);
    return 0;

out_unmap_rdist:
    for (i = 0; i < nr_redist_regions; i++)
        if (rdist_regs[i].redist_base)
            iounmap(rdist_regs[i].redist_base);
    kfree(rdist_regs);
out_unmap_dist:
    iounmap(dist_base);
    return err;
}
```

函数gicv3_of_init如上所示，该函数将映射GIC寄存器基地址，然后获取GIC的版本信息(通过读GICD_PIDR2寄存器，读取寄存器bit[7:4]，是0x3就是GIC v3，前面小节GIC寄存器有描述)， 获取设备树的属性值，调用gic_init_bases函数

```c
// 内核源码/drivers/irqchip/irq-gic-v3.c

static int __init gic_init_bases(void __iomem *dist_base,
            struct redist_region *rdist_regs,
            u32 nr_redist_regions,
            u64 redist_stride,
            struct fwnode_handle *handle)
{
    u32 typer;
    int gic_irqs;
    int err;

    if (!is_hyp_mode_available())
        static_branch_disable(&supports_deactivate_key);

    if (static_branch_likely(&supports_deactivate_key))
        pr_info("GIC: Using split EOI/Deactivate mode\n");

    /*对GIC v3硬件设备相关的数据结构初始化*/
    gic_data.fwnode = handle;                      //一些回调方法
    gic_data.dist_base = dist_base;                //Distributor内存区间的地址
    gic_data.redist_regions = rdist_regs;          //gic中Redistributor域的信息
    gic_data.nr_redist_regions = nr_redist_regions; //Redistributor域的个数
    gic_data.redist_stride = redist_stride;         //Redistributor域之间的步长

    /*
    * Find out how many interrupts are supported.
    * The GIC only supports up to 1020 interrupt sources (SGI+PPI+SPI)
    */
    typer = readl_relaxed(gic_data.dist_base + GICD_TYPER);
    gic_data.rdists.gicd_typer = typer;
    gic_irqs = GICD_TYPER_IRQS(typer); //获取bit[4:0]的值，算出SPI个数
    if (gic_irqs > 1020)
        gic_irqs = 1020;
    gic_data.irq_nr = gic_irqs;

    /*在系统中为GIC V3注册一个irq domain的数据结构*/
    gic_data.domain = irq_domain_create_tree(handle, &gic_irq_domain_ops,
                        &gic_data);
    irq_domain_update_bus_token(gic_data.domain, DOMAIN_BUS_WIRED);
    gic_data.rdists.rdist = alloc_percpu(typeof(*gic_data.rdists.rdist));
    gic_data.rdists.has_vlpis = true;
    gic_data.rdists.has_direct_lpi = true;

    if (WARN_ON(!gic_data.domain) || WARN_ON(!gic_data.rdists.rdist)) {
        err = -ENOMEM;
        goto out_free;
    }

    gic_data.has_rss = !!(typer & GICD_TYPER_RSS);
    pr_info("Distributor has %sRange Selector support\n",
        gic_data.has_rss ? "" : "no ");

    if (typer & GICD_TYPER_MBIS) {
        err = mbi_init(handle, gic_data.domain);
        if (err)
            pr_err("Failed to initialize MBIs\n");
    }

    set_handle_irq(gic_handle_irq);  //设置中断回调函数，gic_handle_irq将查表等获取到是哪一个中断，处理中断

    gic_update_vlpi_properties();   /*更新Redistributor相关的属性*/

    if (IS_ENABLED(CONFIG_ARM_GIC_V3_ITS) && gic_dist_supports_lpis())
        its_init(handle, &gic_data.rdists, gic_data.domain);  //初始化ITS

    gic_smp_init();    //设置核间通信等
    gic_dist_init();   //初始化Distributor，
    gic_cpu_init();
    gic_cpu_pm_init(); //初始化GIC电源管理

    return 0;

out_free:
    if (gic_data.domain)
        irq_domain_remove(gic_data.domain);
    free_percpu(gic_data.rdists.rdist);
    return err;
}
```

gic_init_bases函数如上，一些解释看下上面的代码注释，源码分析到这，主要分析了下中断控制器驱动相关源码，详细的IRQ Domain映射，irq事件详细处理过程可自行参考内核源码。