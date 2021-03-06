# BSP中的Kconfig文件说明

标签： RT-Thread规范

---

自从RT-Thread 3.0.0开始，BSP中推荐使用menuconfig方式来进行系统配置，同时包管理器也附着在menuconfig上。所以RT-Thread 3.0.x后续的版本，[env工具][1]变成一份必不可少的工具。

本篇文档主要说明BSP包中和Kconfig相关的部分。

## Kconfig 语法

Kconfig源自Linux Kernel，是Linux Kernel的典型、成熟配置方式，也广泛被其他开源项目所使用，[例如buildroot][2]，Kconfig的完整语法放置于[Linux Kernel][3]中。

## BSP中的Kconfig文件说明

一份BSP中典型的Kconfig文件如下所示

```Kconfig
mainmenu "RT-Thread Project Configuration"

config $BSP_DIR
    string
    option env="BSP_ROOT"
    default "."

config $RTT_DIR
    string
    option env="RTT_ROOT"
    default "../.."

config $PKGS_DIR
    string
    option env="PKGS_ROOT"
    default "packages"

source "$RTT_DIR/Kconfig"
source "$PKGS_DIR/Kconfig"
source "$BSP_DIR/drivers/Kconfig"

config SOC_IMXRT1052
    bool 
    select ARCH_ARM_CORTEX_M7
    default y

```

其中，

* BSP_DIR变量定义了BSP根目录，默认是`.` （因为上面一份Kconfig文件放置于BSP板级支持包目录下），除非Host系统中定义了`BSP_ROOT`的环境变量；
* RTT_DIR变量定义了RT-Thread **根目录**，因为板级包目录默认放置在`rt-thread/bsp`目录下，所以这个变量的默认值是`../..`，除非系统中定义了`RTT_ROOT`的环境变量；
* PKGS_DIR变量定义了RT-Thread包根目录，一般它会从系统的环境变量`PKGS_ROOT`中获得，而如果使用RT-Thread/env工具，env工具在启动console终端时会默认地定义这个环境变量；

这份Kconfig文件的后面部分则把相关的Kconfig文件都包含到这个Kconfig文件中来，最关键的是RT-Thread主干Kconfig文件 `$RTT_DIR/Kconfig` 和包Kconfig文件 `$PKGS_DIR/Kconfig` 。而`$BSP_DIR/drivers/Kconfig`文件则是把驱动的Kconfig配置文件添加进来。

更后面的配置项：

```kconfig
config SOC_IMXRT1052
    bool 
    select ARCH_ARM_CORTEX_M7
    default y
```

它定义了芯片的型号`（SOC_IMXRT1052）`，同时选择了对应的芯片架构`（ARCH_ARM_CORTEX_M7）`。建议BSP的配置文件都做相应的定义，以反映芯片的型号，架构情况。因为在Kconfig的其他配置中，可能会有架构级的依赖项。缺少架构的定义，一些配置可能会变得不可见。

## 驱动中的Kconfig文件

一般建议在`$BSP_DIR/drivers`目录下加入Kconfig配置文件，从而不需要总是修改`BSP_DIR/Kconfig`文件，一份简单的驱动Kconfig如下所示：

```kconfig
config RT_USING_UART0
    bool "Enable UART0"
    default n

config RT_USING_UART1
    bool "Enable UART1"
    default y

config BSP_DRV_CLCD
    bool "CLCD driver"
    depends on RT_USING_GUIENGINE
    default y

config BSP_DRV_EMAC
    bool "EMAC driver"
    depends on RT_USING_LWIP
    default y
```

其中一些需要着重说明的部分：

* **配置项是属于RT-Thread本身的，还是属于这个BSP所特有的？**
    - 如果是RT-Thread本身的，那么配置项应该放在RT-Thread中，而不是BSP目录下；<如果RT-Thread确实缺少对应的一些配置项，请向RT-Thread github提交PR>
    - 如果是BSP本身的，则在`$BSP_DIR/Kconfig`文件中添加相应的配置项；如果对应配置项还是默认必须的，那么可直接添加一个无提示、默认是选择的配置项。
* 针对于外设驱动配置项，可以多考虑携带`depends`或`select`的配置项：
    - `depends on OPTION` 表示这个配置项依赖于OPTION，只有当OPTION被选择时，这个配置项才有效；
    - `select OPTION` 表示当当前配置项被选择时，OPTION会被自动选择上；
* 另外，请注意外设配置在`$RTT_DIR/components/drivers/Kconfig`中还存在一份；这份配置可以定义出系统中哪些抽象型的外设驱动框架已经定义了，可以由这份配置来做BSP板级支持包驱动的依赖 `depends on` ；也可以由BSP板级支持包外设驱动来做选择 `select`。

  [1]: https://www.rt-thread.org/page/download.html
  [2]: https://buildroot.org/
  [3]: https://www.kernel.org/doc/Documentation/kbuild/kconfig-language.txt
