Linux4.4 以后引入了动态设备树 (Dynamic DeviceTree)，我们这里翻译为“设备树插件”。

设备树插件可以理解为主设备树的“补丁”它动态的加载到系统中，并被内核识别。

例如我们要在系统中增加 RGB 驱动，那么我们可以针对 RGB 这个硬件设备写一个设备树插件，然后编译、加载到系统即可，无需重新编译整个设备树。

设备树插件是在设备树基础上增加的内容，我们之前讲解的设备树语法完全适用，甚至我们可以直接将之前编写的设备树节点复制到设备树插件里。具体使用方法介绍如下。

## 1 设备树插件格式

设备树插件拥有相对固定的格式，甚至可以认为它只是把设备节点加了一个“壳”编译后内核能够动态加载它。格式如下，具体节点省略。

目录：kernel/arch/arm64/boot/dts/rockchip/overlay/rk356x-lubancat-pwm7-ir-overlay.dts

```C
/dts-v1/;
/plugin/;

/ {
        compatible = "rockchip,rk3568";

        fragment@0 {
                target = <&pwm7>;
 
                __overlay__ {
                        status = "okay";
                };
        };
};
```

- `/dts-v1/;` 和 `/plugin/;`：这些是 Device Tree 的头部声明，指定了使用的 Device Tree 版本和插件。
    
- `compatible = "rockchip,rk3568";`：这里指定了设备树描述的硬件平台
    
- `fragment@0 { ... };`：这是一个设备树片段，用于描述特定硬件组件的配置。
    
- `target = <&pwm7>;`：这里指定了片段应用的目标，即设备树中的 PWM7 控制器。
    
- `overlay { ... };`：这是一个特殊的语法，表示将以下配置作为设备树的覆盖层应用到目标节点上。
    
- `status = "okay";`：这行指定了 PWM7 控制器的状态为 "okay"，表示设备树启用了这个 PWM 控制器。
    

  

## 2 编译

修改内核目录/arch/arm64/boot/dts/rockchip/overlays 下的 Makefile 文件，添加我们编辑好的设备树插件。并把设备树插件文件放在和 Makefile 文件同级目录下。以进行设备树插件的编译。

```C
// 加载配置文件  
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- lubancat2_defconfig  

// 使用 dtbs 参数单独编译设备树  
make ARCH=arm64 -j4 CROSS_COMPILE=aarch64-linux-gnu- dtbs  
```

编译出来的设备树插件在内核源码/arch/arm64/boot/dts/rockchip/overlay/

lubancat-led-overlay.dtbo，将设备树插件传到板卡的 /boot/dtb/overlay/ 目录下，

并在 /boot/uEnv/uEnv.txt 按照格式添加我们的设备树插件，然后重启开发板，那么系统就

会加载我们编译的设备树插件。

  

## 3 设备树插件功能宏

---

```C
Device Drivers → Device Tree and Open Firmware support → Device Tree overlays
```

```C
CONFIG_OF_OVERLAY=y
```

以及：

![](https://ncn13z89mqjm.feishu.cn/space/api/box/stream/download/asynccode/?code=MWJhZjZjOTk4NzllYTYyNTIwMThiMzdjMzJjOTI3YmJfVXRYNnpsZjhPT2ozaWxlenBrdFFLc3dWV1lWV2ZxNDZfVG9rZW46U1ZRZWJzb1djb3JUdFN4M0Q5a2NnT21vbkxiXzE3NDk2Mjk2Njg6MTc0OTYzMzI2OF9WNA)

```C
OVERLAY_FS
OVERLAY_FS_REDIRECT_DIR
OVERLAY_FS_REDIRECT_ALWAYS_FOLLOW
OVERLAY_FS_INDEX
OVERLAY_FS_XINO_AUTO
OVERLAY_FS_METACOPY
```

  

## 4 configfs

/sys/kernel/config/

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/fd415af2d1532e8a4d055018d1b0de6b.png)
