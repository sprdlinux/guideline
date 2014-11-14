 Sprdlinux Programming Guideline
=====

目录  (Content)
-----
[TOC]

 前言
-----
编程是一门语言艺术——不开玩笑。
程序强调理性逻辑，语言讲究感性表达。
逻辑不正确，程序无法运行；表达不优雅，文章难成佳作。

计算机编程语言诞生于上世纪60年代，历史不足百年，但数百万程序员笔耕不缀，汗牛充栋。Linux内核代码历时20多年，已达千万行之巨；sprd kernel 近2年的提交，累计也已有100多万行。
我经历过一个公司，早期只追求程序运行结果，而忽视代码表达，还说什么“多快好省，坏了再整”——可后来负责“坏了再整”的那些人，没有一个不边看代码边骂娘的。

编程有如行文，又胜过行文，不仅要结构清晰，逻辑正确，而且要表达优雅，简洁易懂。
外在和内在巧妙融合，理性和感性完美统一，当事物达到这种境界，人们就会称之为艺术。

 命名 (Naming)
-----
古语云：名不正，则言不顺；言不顺，则事不成。
虽然是讲治世之道，但细细想来，似乎和编程规律也隐约相通。
Linux kernel集万众之力，成万千代码。目录名、文件名、函数名、变量名、宏名，乃至作者名，如果各自为政，恐怕早就一团乱麻了。
综合各方分析，采众家之长，Sprdlinux社区版约定如下：

### 公司名 (Manufacturer Name)

>- 全称 **"Spreadtrum"**
>- 缩写 **"sprd"**

全称仅在注释、描述、介绍中提及公司时使用。
文件、变量、函数中只使用缩写，宏名按惯例使用大写 **“SPRD”**

> 在现有代码中存在sc的使用，如mach-sc。
> sc是spreadtrum芯片前缀，如sc7731。
> 作为后来者，sc极易和upstream中的既有名称混淆重复。

### 平台名 (Family Name)

我司芯片除ARM核部分，SoC架构基本相似，均源于shark.
名称纷繁的原因多源于市场策略和通信制式的不同。而按SoC惯例，是以cpu为视角来划分。
因此缩减平台名称如下:
>- **tiger**: Cortex-A5MP2 32bit架构
>- **shark**: cortex-A7MP4 32bit架构(scx30)，含原shark, tshark,  pike 
>- **sharkl**: cortex-A7MP4 32bit 或 cortex-A53 64bit架构(scx35)，含原sharkl, tsharkl(sharkl64).
>- **whale**:  cortex-A53MP8 Big-LITTLE架构

-  _争议_：upstream曾存在一个平台也叫做shark, 目前已被移除。
但多数情况下，不要安排SHARK单独使用，而是和SPRD组合，如： **ARCH_SPRD_SHARK**
也不要设计这样的目录：mach_shark，因为shark,sharkl都包含在mach-sprd里。
可以避免重名的问题。

- _关于scx_: 现有平台命名有使用“scx+数字”的方式。
	- 这种命名无任何官方依据
	- 和芯片型号或者手机型号组合后看上去很罗嗦： sprd_scx35l_sc9630.dts

### 芯片名 (Chip Name)

>- **主力**芯片型号 **sc####**

因我司芯片型号较为灵活，因此**引脚兼容**、**配置相同**的芯片统一用最初命名的型号代表，称之为该型号兼容。也就是说，没有重新流片的芯片，无论remark成什么型号，都使用最初的型号代表。
比如： sc1234和sc1155仅仅因为销售策略而型号不同，因为sc1234最先使用，sc1155称之为sc1234 compatible，不再另行命名。

### 板名 (Board Name)

>- 手机型号**sp####xxx[-....]**

如 sp7730gga；如果同一个型号又有区分，则在后部添加描述，如sp7731gga-openphone, sp7731gga-customer等。

### 目录名 (Directory Name)

以下相对于kernel源代码目录树根目录:

- 包含公司名、平台名、芯片名、板名的目录，不应位于第一、二级目录（firmware目录除外）
-  公司名(**sprd**)应用作独立的目录名用以和其他厂商区分，不应和功能连体，尤其是一大堆并列，如当前tsharl_re分支中的drivers/media/下：

**Looks BAD:**
>   ./drivers/media/**sprd_gsp**
>   ./drivers/media/**sprd_dma_copy**
>    ./drivers/media/**sprd_rotation**
>    ./drivers/media/**sprd_isp**
>    ./drivers/media/**sprd_dcam**
>    ./drivers/media/**sprd_sensor**
>    ./drivers/media/**sprd_scale**
>    ./drivers/misc/**sprd_vsp**
>    ./drivers/misc/**sprd_cproc**
>    ./drivers/misc/**sprd_srt**
>    ./drivers/misc/**sprd_2351**
>    ./drivers/misc/**sprd_jpg**
>    ./drivers/misc/**sprd_otp**

**Feel better?**
>    ./drivers/media/**sprd**/dma_copy
>    ./drivers/media/**sprd**/rotation
>    ./drivers/media/**sprd**/isp
>    ./drivers/media/**sprd**/dcam
>    ./drivers/media/**sprd**/sensor
>    ./drivers/media/**sprd**/scale
>    ./drivers/misc/**sprd**/vsp
>    ./drivers/misc/**sprd**/cproc
>    ./drivers/misc/**sprd**/srt
>    ./drivers/misc/**sprd**/2351
>    ./drivers/misc/**sprd**/jpg
>    ./drivers/misc/**sprd**/otp

- ARM32体系架构目录：arch/arm/**mach-sprd/**
- 驱动目录： drivers/[ sub-systems ]/**sprd/**

### 文件名 (File Name)

#### **.dts/.dtsi文件**
基本格式:  公司名 + [ 平台名 | 芯片名 ] + [ 板名 ].[ dtsi | dts ]
 
> 例如：
> ![dt-files- relationship](https://raw.githubusercontent.com/sprdlinux/guideline/master/images/dt-relations.png)

**sprd-shark.dtsi**：描述shark平台所共有的硬件资源，如cpu, interrupt-controller, uarts, etc.
**sprd-sc7730.dtsi** ：首先包含sprd-shark.dtsi中的内容，描述(override) sc7730这款芯片的差异，比如cp的配置资源等。
**sprd-sp7730gga-openphone.dts**：代表基于上述芯片的最终手机成品，在包含上述2个文件的基础上，补充描述如内存配置，pinctrl-mux，端口使能，以及在板外设等。 

#### .c/.h/.S文件

基本格式:  [ 公司名 | 平台名 ] + 功能 + [ 芯片名 | 板名 ].[ c | h | S ]

-  功能与体系架构无关，或存在于所有平台，且又为我司独有，使用 **公司名+功能**
> 例:  sprd-debug.c

- 功能仅存在于某一平台中，使用 **平台名+功能**
> 例:  sound/soc/sprd/shark-codec.c

- 功能仅存在于某一芯片，使用 **功能+芯片名**
> 例：drivers/pinctrl/pinctrl-sc7730.c

- 片外外设或片内外设IP非sprd独有，应按其芯片型号或厂商名称独立命名；如kernel upstream中已有该厂商文件，应遵从其模式，不应在sprd目录下另行划分。

- 特别的，如某子系统有其特有文件命名惯例，则遵循其惯例。

- 原则上，当很多名字接近文件在同一目录时，重复的字段应优先用作前缀。
> 例： 
>  good              |  bad
>  :------------------|  :--------------
>  pinctrl-sc7730.c   | sc7730-pinctrl.c
>  pinctrl-sc7731.c   | sc7713-pinctrl.c

### 函数名 (Function Name)

- 函数名均由小写字母、下划线构成，禁止花体、驼峰体、大小写混排、以及匈牙利命名法。

- 函数名应望文知意，简洁易懂，不宜使用过长或过于生癖的单词。适当使用缩写，但缩写形式如有惯例应遵循，不宜自创。

- 模块内函数如无需其他模块引用，应定义为static类型；导出函数(EXPORT)应加**sprd_**前缀，以防污染命名空间。

- 以下划线'__'开头的函数名仅在确有必要时使用。常见情况是用来实现对同一函数多种调用方式，比如实现缺省参数；或是表示和体系架构相关的底层实现，比如__raw_readl()。

### 变量名 (Variable Name)

- 变量名规则与函数命名规则相同，但变量名应普遍短于函数名。

- 局部变量尽量使用一个单词达意。但同时尊重命名习惯，比如常用i,j命名for循环变量。

### 宏名 (Macro Name)

- 通常使用全部大写字母和下划线构成，对函数的宏重命名使用小写。

- 名称选取和函数一致，简洁易懂为宜。

- Kconfig宏应联用大写的 **公司名 + [ 平台名 | 芯片名 | 板名 ]** 做区分。

> 例1：公司所有平台共有 **ARCH_SPRD**，**SPRD_MEM_POOL**
> 例2：某一平台特有 **ARCH_SPRD_SHARK**，**SPRD_SHARK_DMC_FREQ**
> 例3：某一芯片特有 **ARCH_SPRD_SC7730**，**SOUND_SPRD_SC7730**
> 例4：某型号手机或某类型板卡特有： **MACH_SPRD_SP7730GGA**，**MACH_SPRD_OPENPHONE** 

- 寄存器相关的宏定义应使用与芯片spec描述一致的名称，便于检索。

- 模块内部使用的宏，无需使用SPRD前缀，避免重复罗嗦。如串口驱动模块中对寄存器偏移量的宏定义。

### 作者名 (Author Name)

- 名在前，姓在后，首字母大写，中间空格分开。比如Orson Zhai，而非orson.zhai，OrsonZhai或zhai orson。

- 原则上使用护照上姓名的汉语拼音，应避免使用中文，因为可能会在某些终端下显示为乱码。

文件布局 ( Layout )
-----

兵法讲谋定而后动，编程亦如是。尤其是大型项目，文件布局有如派兵布阵，是项目架构设计的直接体现。文件目录层次安排不是简单将文件归类，而是与整体架构模型关系密切。
Linux kernel的目录层次总体上简洁而清晰，但arch/arm/下的布局曾一度混乱不堪。幸运的是，被Linux掌门人Linus Tovalds毒舌批评之后，ARM kernel社区开始了基于dt(device tree)的大幅度清理，效果立竿见影。
因此本文不再探讨非dt情况——他们都该被扫入历史垃圾堆。

### 硬件描述

    {kernel_path}/arch/arm/boot/dts/
    {kernel_path}/arch/arm64/boot/dts/
    
所有ARM架构的SoC芯片, 板级相关的硬件描述文件都放于此处，包括dts和dtsi。
dts为板级描述，dtsi为SoC或family描述。

    {kernel_path}/arch/arm/boot/dts/include/dt-bindings
    -> {kernel_path}/include/dt-bindings/
  
 和硬件描述相关的头文件，与dtsi不同之处在于其内容多为对dt binding值的宏定义。
 
    {kernel_path}/Documentation/devicetree/bindings/
    
 描述和介绍binding用法用例的文档，必须和dt文件一同提交。
 
### 平台初始化

    {kernel_path}/arch/arm/mach-sprd/

放置32位ARM初始化文件。
ARM64已经取消了这个目录，可以想见arm32之所以保留也是照顾历史兼容性而已。
除了DT_MACHINE_START宏和极少数初始化代码，大部分初始化工作应被分解到drivers和bootloader中。

### 驱动模块

    {kernel_path}/drivers/
    ├── base
    │   ├── power
    │   └── regmap
    ├── block
    │   └── zram
    ├── bluetooth
    ├── bus
    ├── clk
    ├── clocksource
    ├── cpufreq
    ├── cpuidle
    │   └── governors
    ├── devfreq
    ├── dma
    ├── dma-buf
    ├── gpio
    ├── gpu
    │   ├── drm
    ├── hid
    │   ├── i2c-hid
    │   └── usbhid
    ├── hwmon
    │   └── pmbus
    ├── hwspinlock
    ├── i2c
    │   ├── busses
    │   └── muxes
    ├── idle
    ├── iio
    │   ├── adc
    │   ├── common
    │   │   ├── hid-sensors
    │   ├── dac
    │   ├── gyro
    │   ├── light
    │   ├── magnetometer
    │   ├── orientation
    │   ├── temperature
    │   └── trigger
    ├── input
    │   ├── keyboard
    │   ├── misc
    │   ├── mouse
    │   ├── tablet
    │   └── touchscreen
    ├── iommu
    ├── irqchip
    ├── leds
    │   └── trigger
    ├── mailbox
    ├── media
    │   ├── common
    │   ├── i2c
    │   │   └── soc_camera
    │   ├── platform
    │   │   ├── soc_camera
    │   ├── radio
    │   ├── tuners
    │   ├── usb
    │   └── v4l2-core
    ├── memory
    ├── mfd
    ├── misc
    ├── mmc
    │   ├── card
    │   ├── core
    │   └── host
    ├── mtd
    │   ├── chips
    │   ├── devices
    │   ├── lpddr
    │   ├── maps
    │   ├── nand
    │   ├── spi-nor
    │   └── ubi
    ├── net
    │   ├── wireless
    ├── nfc
    ├── oprofile
    ├── phy
    ├── pinctrl
    ├── platform
    ├── power
    │   ├── avs
    │   └── reset
    ├── powercap
    ├── pwm
    ├── rapidio
    │   ├── devices
    │   └── switches
    ├── regulator
    ├── reset
    ├── rtc
    ├── soc
    ├── spi
    ├── thermal
    ├── tty
    │   ├── serial
    ├── usb
    ├── vhost
    ├── video
    │   ├── backlight
    │   ├── console
    │   ├── fbdev
    │   │   ├── core
    │   └── logo
    ├── virt
    ├── virtio
    ├── watchdog


- 对应与dt硬件描述中的节点，每个驱动文件各归其位。
- 对于soc特有一些驱动，可考虑新近增加的 drivers/soc目录。
- 实在找不到合适目录放置的驱动，比如ion，可参考drivers/staging/目录，自行划分一个android目录存放。一旦社区决定最终的位置，应按upstream修改。

### 头文件

    {kernel_path}/include/

此目录用于存放需要跨模块引用的函数声明、宏定义等头文件。
换言之，如果不是暴露给整个内核调用的公共函数，不要在这里放置。
模块私有的头文件，应该和各模块c文件放在一起。

### 固件

    {kernel_path}/firmware/
    
有些固件没有开放代码，无法通过编译得到。
这个目录并不是正式存放固件的位置（不符合GPL约定）。
但没有这些固件，某些功能将无法使用。
作为权宜之计，我们可以把固件二进制文件放于此处。

DT描述 ( Device Tree ) 
-----

### 规则
### 节点
### 分层
### override
### compatible string
### Interrupt controller
### clock
### power management

子系统 ( Sub-system & Module )
-----

### 基本输出 ( stdout )
### pinctrl
### 时钟
### 中断
### 内存控制
### DMA
### 电源管理
#### PSCI
### Framebuffer
### 按键和LED
### 多媒体
### 全局寄存器问题
### sipc

ARM64兼容
-----

### 初始化
### DT
### ABI

工具 ( Tools )
-----

社区 ( Community )
-----





--
> Created by Orson Zhai (orson.zhai@gmail.com)
> Written with [StackEdit](https://stackedit.io/).