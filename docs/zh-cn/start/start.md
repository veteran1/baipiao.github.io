## 准备工作



如果你是一个新手，请预先准备好以下工具，如果你不知道怎样获取，请加我们的技术支持群`1001220381`，在群文件中下载。



> * 8051-ELL库源码
> * STC-ISP助手
> * Keil4 v901或 Keil5
> * git（非必须）



### 下载源码

<br>



下面以Gitee仓库下载代码作为演示，Github类似。



* [Gitee仓库](https://gitee.com/zeweni/ELL-8051-LIB) ，下载速度快，但需要注册账户。

* [Github仓库](https://github.com/zewen-i/8051-ELL-LIB) ，下载速度慢，但不需要注册账户。



![](image\gitee下载源码.png)


如果你使用git工具，那么更新库将会变得非常方便，你只需要在一个文件夹中右键打开控制台，然后输入：

```
git clone https://gitee.com/zeweni/ELL-8051-LIB.git
```

即可完成下载。

<br>

### ELL库目录结构



先简单看一下目录结构，方便你了解整个库的框架。

<br>



| 一级目录  |  二级目录   |        描述         |
| :-------: | :---------: | :-----------------: |
|    doc    |     ...     |    一些文档资料     |
| examples  |             |      示例工程       |
|           |   STC8Ax    | STC8A系列的示例代码 |
|           |   STC8Cx    | STC8C系列的示例代码 |
|           |     ...     |      其他型号       |
| libraries |             |      ELL库文件      |
|           |    core     |  寄存器和启动文件   |
|           | components  |       组件库        |
|           | peripherals |  芯片的片内外设库   |
|           |   drives    |     设备驱动库      |
|  project  |             |      模板工程       |
|           |   STC8Ax    | STC8A系列的模板工程 |
|           |   STC8Cx    | STC8C系列的模板工程 |
|           |     ...     |      其他型号       |

<br>



## 编译代码



### 熟悉工程框架



我们下载好ELL以后，先解压一下。之后打开project文件夹，以STC8Hx为例，点击Keil工程。



> ELL库的Keil工程默认进行了几项配置：

- 勾选了LX51编译，并在`LXMisc`中配置了REMOVEUNUSED宏
- 代码烧录文件路径在工程根目录的`build`文件夹



因此，对于没有被调用的函数，Keil是不会编译的，这样更方便我们做库开发。

<br>

同时，在大家打开工程以后，需要先关注一下左边的目录树：

![](image\Keil_Project.png)

```
|--lib/startup:启动文件分支，在这里你可以做一些传统Keil对51启动前的一些配置，同时还包括ELL库的硬件仿真设置和对RTOS的支持
	|--startup_stc8h.s
|--lib/peripherals:片内外设库分支：和本型号相关的一些外设模块
	|--...
|--lib/components:组件库分支：提供好用的组件，比如TMT任务框架、精准延时组件等
	|--...
|--lib/drives:设备驱动库分支
	|--...
|--user/main: main分支：对于系统的初始化和中断操作
    |--Lib_CFG.h	ELL库配置头文件
    |--main.c	main函数入口文件
    |--init.c   系统初始化文件
    |--isr.c    中断服务函数文件
 |--user/app 应用分支：开发者自主编写的代码放在这里面
```



熟悉了目录分支，可以帮你快速了解整个ELL库的框架。



### 配置ELL



分两步走，一个是对启动文件的配置，一个是对Lib_CFG.h的配置。



#### 配置启动文件



startup文件，我们只需要关注硬件仿真的配置，你可以按住**ctrl+F**，输入`DEBUCTRL`，或者`硬件仿真`，然后按下**Find next**直接跳转到对应位置。你会看到如下代码：



```C
;-------------------------------------------------------------------------
;  配置硬件仿真需要占用的XDATA空间
;  值得注意的是，这里的设置主要是影响堆栈和可重入函数。
;  如果你想真正指明内存大小，请在Keil的配置页面Target右下方填写设置。
;  XDATA space for hardware emulation configuration
;  The length of XDATA memory in bytes.
;  It is worth noting that the settings here mainly affect the stack and 
;  reentrant functions.
;  If you really want to indicate the memory size, please fill in the 
;  settings at the bottom right of target on keil's configuration page.
;
;  如果你不需要硬件仿真，那么你可以忽略这部分代码。
;  If you don't need hardware emulation, 
;  you can ignore this part of the code.
;-------------------------------------------------------------------------

; @name    DEBUCTRL
; @brief   硬件仿真占用XDATA空间的开关。
;          Hardware simulation takes up XDATA space switch..
; @Note    如果你不配置它，编译器将不会为你预留出仿真需要的XDATA内存空间。
;          If you don't configure it, the compiler won't reserve 
;          XDATA memory for you.

DEBUCTRL        EQU     1   ; Set to 1 to reserve XDATA space for hardware simulation.


; @name    DEBUGXDATALEN
; @brief   硬件仿真debug需要占用XDATA的大小.
;          Hardware simulation debug needs to occupy the size of XDATA.
; @Note    这里设置denbug占用XDATA内存的字节长度。
;          一般是最后768个字节，如果有不同，请自己根据STC-ISP助手上的说明进行修改。
;          Set the byte length of XDATA memory occupied by debug.
;          Generally, it is the last 768 bytes. If it is different, 
;          please modify it according to the instructions on stc-isp assistant.

DEBUGXDATALEN    EQU    300H   
```



>  DEBUCTRL宏：是硬件仿真的开关，写1使能这个功能
>
> DEBUGXDATALEN宏：是为硬件仿真预留的XDATA空间大小，位置为XDATA末尾的部分



关于至少应该预留多少空间，你可以在STC-ISP助手查看：

![](image\STC-ISP硬件仿真设置.png)



**值得关注的是，我们对启动文件的操作，除了配置硬件仿真，还会对函数可重入的相关设置产生影响，但这里不展开讲，请读者留意。**



如果不需要函数可重入，不用关心这里的问题，并且ELL库做好了相关工作。



#### 配置Lib_CFG.h文件



对于这个头文件，基本不需要太多的配置，你只要关注两个地方，选择芯片型号和配置系统时钟宏。

```c
#define    STC8Ax      (0)
#define    STC8AxD4    (1)
#define    STC8Cx      (2)
#define    STC8Fx      (3)
#define    STC8Gx      (4)
#define    STC8Hx      (5)

/**
 * @brief		配置ELL库MCU选型。
 * @details	    Configure ADC chip internal  peripheral.
**/
#define    PER_LIB_MCU_MUODEL     STC8Hx


/**
 * @brief		配置系统时钟管理片内外设，没有中断相关的控制宏。
 * @details	    Configure the system clock for time calculation.
**/
#define   PER_LIB_SYSCLK_CTRL        (1) /*!< 系统时钟模块控制宏，写1开启，写0关闭。*/

#define   PER_LIB_SYSCLK_INIT_CTRL   (1) /*!< 系统时钟模块初始化相关宏，写1开启，写0关闭。*/
#define   PER_LIB_SYSCLK_WORK_CTRL   (1) /*!< 系统时钟模块工作相关宏，写1开启，写0关闭。*/
#define   PER_LIB_SYSCLK_VALUE     (0UL) /*!< 等于0,自动获取系统时钟 */

```



> PER_LIB_MCU_MUODEL宏：根据你的芯片型号进行选择
>
> PER_LIB_SYSCLK_VALUE宏：设置为0，自动获取系统时钟



要提醒的是，如果你使用的是外部晶振，这里的宏一定要设置成，和你外部晶振一样的工作频率，否则MCU不会正常工作。

同时，如果你使用了系统初始化函数，对时钟进行了操作，那么这里的频率，也要和你**代码最终配置的系统频率**一样才行。



新手建议采用默认的配置即可。



> ELL库默认的配置（main/Lib_CFG.h）：

* 采用ELL自动获取系统时钟频率（仅限于内部IRC时钟）

* 开启了精准延时组件、printf重定向组件



#### 初步了解ELL库函数



我们打开init.c文件，看一下STC8x_System_Init函数。

```c
/**
 * @brief   MCU初始化函数
 * @details MCU initialization function
 * @param   None
 * @return  None
**/
void STC8x_System_Init(void)
{
	DELAY_POS(); /*!<上电延时一段时间。 Power on stability delay. */	
	STC8x_SYSCLK_Config(); /*!<初始化系统时钟。 Initialize system clock. */
	delay_init(); /*!<初始化精准延时组件。 Initialize the precision delay component. */
	
	STC8x_GPIO_Config(); /*!<初始化GPIO。 Initialize GPIO. */
	STC8x_UART_Config(); /*!<初始化串口。 Initialize UART. */
	STC8x_TIMER_Config(); /*!<初始化定时器。 Initialize TIMER. */
	
	/*
		Add hardware driver initialization code here.
	*/
	
	NVIC_GLOBAL_ENABLE();  /*!< 开启总中断。 Turn on total interrupt. */
}
```



先看STC8x_SYSCLK_Config函数，选中它，按下`F12`跳转。

```C
/**
 * @brief   SYSCLK初始化函数
 * @details SYSCLK initialization function
 * @param   None
 * @return  None
**/
static void STC8x_SYSCLK_Config(void)
{
    /** 首先定义一个系统时钟初始化结构体句柄，并初始化为0，初始化的动作不能少！ */
	SYSCLK_InitType SYSCLK_InitStruct={0}; /*!< Declare structure */

	SYSCLK_InitStruct.MCLKSrc = AUTO;  /*!< 选择AUTO，通过STC-ISP助手设置频率。 */
    
    /** 下面四行代码是代码手动配置的例子，这里不用管 */
	// SYSCLK_InitStruct.IRCBand = IRC_Band_20MHz;
	// SYSCLK_InitStruct.IRCTRIM = 150;
    // SYSCLK_InitStruct.LIRCTRIM = TRIM1;
	// SYSCLK_InitStruct.MCLKDiv = 0;
    
	SYSCLK_InitStruct.SCLKDiv = 0; /* if SCLKDiv = 0, Not output */
	SYSCLK_InitStruct.SCLKOutPin = SCLK_OUT_P16;  
	SYSCLK_Init(&SYSCLK_InitStruct);   /*!< 调用系统初始化函数。 */
	
}
```



> 这里你可能会疑惑，我们已经在Lib_CFG.h配置过宏了，为什么还要在这里初始化系统时钟。原因很简单，宏是对ELL的全局配置，它影响的是时钟相关的计算工作，比如计算串口波特率。而系统时钟初始化函数，是对时钟配置的具体代码实现。两者缺一不可，相辅相成。



我们再来看一下定时器初始化函数：

```C
/**
 * @brief   定时器初始化函数
 * @details TIMER initialization function
 * @param   None
 * @return  None
**/
static void STC8x_TIMER_Config(void)
{
    /** 首先定义一个定时器初始化结构体句柄，并初始化为0，初始化的动作不能少！ */
	TIMER_InitType TIMER_InitStruct={0};
	
	TIMER_InitStruct.Mode = TIMER_16BitAutoReload; /*!< 选择16位自动重装载模式。 */
	TIMER_InitStruct.Time = 1000;     /*!< 定时1ms，单位是us。不需要手动计算，是不是很方便！ */
	TIMER_InitStruct.Run = ENABLE;    /*!< 使能运行控制位。 */
	TIMER0_Init(&TIMER_InitStruct);   /*!< 调用定时器初始化函数。 */
	NVIC_TIMER0_Init(NVIC_PR0,ENABLE); /*!< 调用定时器中断初始化函数。 */
}
```



我想你已经注意到了，配置定时器的时间，不再需要我们手动计算，ELL以及帮你计算好了，即使你改变了系统时钟频率，这里的时间也不会改变！你只需要设置好Lib_CFG.h中`PER_LIB_SYSCLK_VALUE`宏就好。



我们在初始化了定时器中断，那么再带大家看看中断服务函数，打开isr.c文件：

```C
/**
 * @brief   定时器0中断服务函数
 * @details MCU TIMER0 Interrupt request service function
 * @param   None
 * @return  None
**/
void TIMER0_ISRQ_Handler(void)
{

}

```



ELL对所有的中断服务函数进行了宏封装，帮你隐藏了中断号，因此不需要额外记忆大量的中断号。

其他常用的中断服务函数名称如下：

```c
/*           
 * TIMER:
 * void TIMER0_ISRQ_Handler(void)                            
 * void TIMER1_ISRQ_Handler(void)
 * void TIMER2_ISRQ_Handler(void)
 * void TIMER3_ISRQ_Handler(void)
 * void TIMER4_ISRQ_Handler(void)
 *
 * UART:
 * void UART1_ISRQ_Handler(void)
 * void UART2_ISRQ_Handler(void)
 * void UART3_ISRQ_Handler(void)
 * void UART4_ISRQ_Handler(void)
 *
 * EXTI:
 * void EXTI0_ISRQ_Handler(void)                             
 * void EXTI1_ISRQ_Handler(void)                             
 * void EXTI2_ISRQ_Handler(void)                             
 * void EXTI3_ISRQ_Handler(void)  
 *
 * ADC:
 * void ADC_ISRQ_Handler(void)
 *
 * COMP:
 * void COMP_ISRQ_Handler(void)
 *
 * PCA:
 * void PCA_ISRQ_Handler(void)
 *
 * LVD:
 * void LVD_ISRQ_Handler(void)
 *
 * PWM:
 * void PWM_ISRQ_Handler(void)
 * void PWM_ABD_ISRQ_Handler(void)
 *
 */
```



接下来再讲一下IO操作：

```C
/**
 * @brief   GPIO初始化函数
 * @details GPIO initialization function
 * @param   None
 * @return  None
**/
static void STC8x_GPIO_Config(void)
{

	GPIO_MODE_WEAK_PULL(GPIO_P4,Pin_All);  /*!< 初始化P4组IO为准双向口（弱上拉）模式 */
	GPIO_MODE_OUT_PP(GPIO_P6,Pin_1|Pin_2); /*!< 初始化P61和P62为推挽输出模式 */
	
}

```



GPIO采用宏函数操作，非常简练紧凑，在编译阶段世界转化为寄存器操作，没有多余的代码冗余。使用宏函数来封装我们经常使用的IO，会大大降低代码的体积，同时提升MCU的执行效率。



> 如果你想了解更多API的使用手册，你可以直接阅读源代码，也可以访问API手册，进行检索和查阅。
>
> [8051-ELL API手册链接](https://8051-ell-api.vercel.app/index.html)



### 编译烧录



之后就是对工程进行一次编译，看看是否有错误。点击Build 或者快捷键F7，对工程进行编译。

![](image\编译.png)



编译完毕，没有警告，在开启了系统定时器、GPIO、串口、定时器、精准延时组件以后，code仅仅只有`2.7KB`，占用非常小！

![](image\编译结果.png)



接下来，就可以使用STC-ISP助手，将ELL编译的hex文件烧录到单片机当中了！



## 硬件仿真



这里我使用STC官方提供的STC8H实验箱进行演示，分三步：



- ELL库配置启动文件
- STC-ISP助手配置硬件仿真
- Keil配置Dubug选项



### 配置启动文件



参照上面对启动文件的配置内容来设置。



### STC-ISP助手设置



![](image\STC-ISP硬件仿真.png)



烧录进去以后，给实验箱进行断电复位，这一步不能少。



### Keil设置Debug



按照下图进行设置：

![](image\Keil-debug设置.png)



设置完毕以后，按下`debug`，即可进行仿真。

![](image\Keil-debug界面.png)



### 常见问题解决



> 一开仿真，程序就自己跑飞怎么办？

答： 启动文件内，没有正确设置硬件仿真的XDATA占用空间长度。



>  不能仿真串口1，怎么回事？

答：串口1的IO被硬件仿真占用了，请将串口1的工作IO切换到别的上，即可正常仿真。



> 为什么有的时候仿真会跑飞？

答：仿真也是占用XDATA的，会受到内存限制，同时要注意我们的代码内，是否有访问到硬件仿真占用的XDATA区。