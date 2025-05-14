
开发思路：
- 知道我们的处理器有几个核
- 中断/资源进程是否可以绑定在某个具体的CPU上
## 1 CPU节点

设备树的 cpus 节点是用于描述系统中的处理器的一个重要节点。 它是处理器拓扑结构的顶层节点， 包含了所有处理器相关的信息。 下面将详细介绍设备树的 cpus 节点的各个方面。
### 1.1 节点结构

cpus 节点是一个容器节点， 其下包含了系统中每个处理器的子节点。 每个子节点的名称通常为 cpu@X， 其中 `X 是处理器的索引号`。 每个子节点都包含了与处理器相关的属性， 例如时钟频率、 缓存大小等。
### 1.2 处理器属性

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/7d36e8085d52800a45ea2aa9d7376bd3.png)

cpu@X 子节点中的属性可以包括以下信息：
1. device_type： 指示设备类型为处理器（"cpu"） 。
2. reg： 指定处理器的地址范围， 通常是物理地址或寄存器地址。
3. compatible： 指定处理器的兼容性信息， 用于匹配相应的设备驱动程序。
4. clock-frequency： 指定处理器的时钟频率。
5. cache-size： 指定处理器的缓存大小。
### 1.3 处理器拓扑关系

除了处理器的基本属性， cpus 节点还可以包含其他用于描述处理器拓扑关系的节点， 以提供更详细的处理器拓扑信息。 这些节点可以帮助操作系统和软件了解处理器之间的连接关系、组织结构和特性

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/c5ff5aa70125e8b1fbd74218dac83b60.png)


- cpu-map 节点： 描述处理器的映射关系， 通常在多核处理器系统中使用。
	- socket 节点： 描述多处理器系统中的物理插槽或芯片组。
	- cluster 节点： 描述处理器集群， 即将多个处理器组织在一起形成的逻辑组。
		- core 节点： 描述处理器核心， 即一个物理处理器内的独立执行单元。
		- thread 节点： 描
		- 述处理器线程， 即一个物理处理器核心内的线程。

这些节点的嵌套关系可以在 cpus 节点下形成一个层次结构， 反映了处理器的拓扑结构。上述这些节点会在后面的小节进行介绍。 一个单核 CPU 设备树和一个四核 CPU 设备树示例如下所示：

**单核 CPU 示例：**
```C
cpus {
    #address-cells = <1>;
    #size-cells = <0>;
    cpu0: cpu@0 {
        compatible = "arm,cortex-a7";
        device_type = "cpu";
        // 其他属性...
    };
}
```

**多核 CPU 示例：**
```C
cpus {
	#address-cells = <1>;
	#size-cells = <0>;
	cpu0: cpu@0 {
		device_type = "cpu";
		compatible = "arm,cortex-a9";
	};
	cpu1: cpu@1 {
		device_type = "cpu";
		compatible = "arm,cortex-a9";
	};
	cpu2: cpu@2 {
		device_type = "cpu";
		compatible = "arm,cortex-a9";
	};
	cpu3: cpu@3 {
		device_type = "cpu";
		compatible = "arm,cortex-a9";
	};
}
```

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/67b5d4f55eab3aaed5a483a37d1580a3.png)


- cpus 节点是一个容器节点， 包含了 cpu0 子节点。 该节点使用了 `#address-cells` 和 `#size-cell` s 属性来指定地址和大小的单元数量。
- cpu0 子节点代表第一个处理器， 具有以下属性：
    - compatible 属性指定了处理器的兼容性信息
    - device_type 属性指示设备类型为处理器。

你可以在此基础上继续添加其他属性来描述处理器的特性， 如时钟频率、 缓存大小等。
## 2 cpu-map、socket、cluster节点

- `cpu-map` 节点是设备树中用于描述`大小核架构处理器的映射关系`的节点之一。 它的父节点必须是 cpus 节点， 而子节点可以是一个或多个 cluster 和 socket 节点。 通过 cpu-map 节点， 可以`定义不同核心和集群之间的连接和组织结构`。

- `socket` 节点用于描述处理器插槽（socket） 之间的映射关系。 每个 socket 子节点表示一个处理器插槽， 可以使用 cpu-map-mask 属性来指定该插槽使用的核心。 通过为每个 socket 子节点指定适当的 cpu-map-mask， 可以`定义不同插槽中使用的核心`。 这样， 操作系统和软件可以了解到不同插槽之间的核心分配情况。

- `cluster` 节点用于描述`核心（cluster） 之间的映射关系`。 每个 cluster 子节点表示一个核心集群， 可以使用 cpu-map-mask 属性来指定该集群使用的核心。 通过为每个 cluster 子节点指定适当的 cpu-map-mask， 可以定义每个集群中使用的核心。 这样， 操作系统和软件可以了解到不同集群之间的核心分配情况。

通过在 cpu-map 节点中定义 socket 和 cluster 子节点， 并为它们指定适当的 cpu-map-mask，可以提供处理器的拓扑结构信息。 这对于操作系统和软件来说非常有用， 因为它们可以根据这些信息进行任务调度和资源分配的优化， 以充分利用大小核架构处理器的性能和能效特性。
## 3 core、thread节点

"core" 和 "thread" 节点通常用于描述处理器核心和线程的配置。 下面是对这两个节点的详细介绍：

- Core 节点用于`描述处理器的核心`。 一个处理器通常由多个核心组成， 每个核心可以独立执行指令和任务。

- Thread 节点用于`描述处理器的线程`。 线程是在处理器核心上执行的基本执行单元， 每个核心可以支持多个线程。

通过使用 Core 和 Thread 节点， 设备树可以准确描述处理器的核心和线程的配置