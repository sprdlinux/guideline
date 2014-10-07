 Sprdlinux Programming Guideline
=====
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
kernel累积了千万行代码，万人共同开发。目录名、文件名、函数名、变量名、宏名，乃至作者名，如果各行其事，恐怕早就一团乱麻了。
Sprdlinux社区版约定如下：

#### 公司名 (Manufacturer Name)

>- 全称 **"Spreadtrum"**
>- 缩写 **"sprd"**

全称仅在注释、描述、介绍中提及公司时使用。
文件、变量、函数中只使用缩写，宏名按惯例使用大写 **“SPRD”**

#### 平台名 (Platform Name)

我司芯片除ARM核部分，SoC架构基本相似，均源于shark.
名称纷繁的原因多源于市场策略和通信制式的不同。而按SoC惯例，是以cpu为视角来划分。
因此缩减平台名称如下:
>- **shark**: cortex-A7MP4 32bit架构，含原shark, sharkl, tshark, scx15, scx35, pike
>- **shark64**: cortex-A53 64bit架构，原名tsharkl
>- **whale**:  cortex-A53/A7 Big-LITTLE架构

#### 芯片名 (Chip Name)

>- **主力**芯片型号 **sc####**

因我司芯片型号较为灵活，因此**引脚兼容**、**配置相同**的芯片统一用最初命名的型号代表，称之为该型号兼容。也就是说，没有重新流片的芯片，无论remark成什么型号，都使用最初的型号代表。
比如： sc1234和sc1155仅仅因为销售策略而型号不同，因为sc1234最先使用，sc1155称之为sc1234 compatible，不再另行命名。

#### 板名 (Board Name)

>- 手机型号**sp####xxx[-....]**

如 sp7730gga；如果同一个型号又有区分，则在后部添加描述，如sp7731gga-openphone, sp7731gga-customer等。

#### 目录名 (Directory Name)

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
>   ./drivers/media/**sprd**/gsp
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

#### 文件名 (File Name)

##### **.dts/.dtsi文件**
基本格式:  公司名 + [ 平台名 | 芯片名 ] + [ 板名 ].[ dtsi | dts ]
 
> 例如：
> ![dt-files- relationship](https://raw.githubusercontent.com/sprdlinux/guideline/master/images/dt-relations.png)

**sprd-shark.dtsi**：描述shark平台所共有的硬件资源，如cpu, interrupt-controller, uarts, etc.
**sprd-sc7730.dtsi** ：首先包含sprd-shark.dtsi中的内容，描述(override) sc7730这款芯片的差异，比如cp的配置资源等。
**sprd-sp7730gga-openphone.dts**：代表基于上述芯片的最终手机成品，在包含上述2个文件的基础上，补充描述如内存配置，pinctrl-mux，端口使能，以及在板外设等。 

##### .c/.h/.S文件

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
> good              |  bad
> :------------------|:--------------
> pinctrl-sc7730.c   | sc7730-pinctrl.c
> pinctrl-sc7731.c   | sc7713-pinctrl.c

#### 函数名 (Function Name)

- 函数名均由小写字母、下划线构成，禁止花体、驼峰体、大小写混排、以及匈牙利命名法。

- 函数名应望文知意，简洁易懂，不宜使用过长或过于生癖的单词。适当使用缩写，但缩写形式如有惯例应遵循，不宜自创。

- 模块内函数如无需其他模块引用，应定义为static类型；导出函数(EXPORT)应加**sprd_**前缀，以防污染命名空间。

- 以下划线'__'开头的函数名仅在确有必要时使用。常见情况是用来实现对同一函数多种调用方式，比如实现缺省参数；或是表示和体系架构相关的底层实现，比如__raw_readl()。

#### 变量名 (Variable Name)

- 变量名规则与函数命名规则相同，但变量名应普遍短于函数名。

- 局部变量尽量使用一个单词达意。但同时尊重命名习惯，比如常用i,j命名for循环变量。

#### 宏名 (Macro Name)

#### 作者名 (Author Name)

- 名在前，姓在后，首字母大写，中间空格分开。比如Orson Zhai，而非orson.zhai，OrsonZhai或zhai orson。
- 原则上使用护照上姓名的汉语拼音，应避免使用中文，因为可能会在某些终端下显示乱码。





--
> Created by Orson Zhai (orson.zhai@gmail.com)
> Written with [StackEdit](https://stackedit.io/).