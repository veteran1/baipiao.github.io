# Open-SkyEye Wiki
![输入图片说明](Skyeye-logo.png)

SkyEye是一款强大的国产嵌入式模拟器，本仓库对其早期的开源版本进行整理，并提供构建编译和安装的教程。对嵌入式软件仿真系统感兴趣的小伙伴，可以 Star我！

## 1.SkyEye简介

SkyEye是一个开源软件（OpenSource Software）项目，中文名字是"天目"。SkyEye的目标是在通用的Linux和Windows平台上实现一个纯软件集成开发环境，模拟常见的嵌入式计算机系统(这里假定"仿真"和"模拟"的意思基本相同)；可在SkyEye上运行μCLinux以及μC/OS-II等多种嵌入式操作系统和各种系统软件（如TCP/IP，图形子系统，文件子系统等），并可对它们进行源码级的分析和测试。

关于开源版本的更多介绍：[点击我跳转](https://www.oschina.net/p/skyeye?hmsr=aladdin1e1)

目前开源版本停留在1.3.6版本，后期版本由浙江迪捷软件科技有限公司开发和维护，[点击我查看SkyEye的最新介绍](https://www.digiproto.com/product/24.html)。



## 2.Docker部署

### 等待更新







## 3.仓库介绍

这是SkyEye的开源文档仓，根据1.3.6版本的代码目录，按照相同的结构，对每一级的模块进行源码分析。第一期工程，计划完成对SkyEye核心库的注释和整理。同时，在仓库顶层Readme文件，会对SkyEye的整体做个介绍。


为了和最新版本的SkyEye做区分，下文我们统一称呼为SkyEye-1.3.6。并且本仓库中提到的所有代码文件路径，均可在`skyeye-code`仓库中找到，因此不再重复提示。





## 4.整体架构


### 架构介绍

SkyEye-1.3.6是一个仿真单板的开发平台，提供了丰富的 API 函数来基于已有的仿真模块进行二次开发。我们可以把整个SkyEye-1.3.6仿真平台分为两大部分，核心库和各种其他外围动态模块。其中外围动态模块大体分类如下：

| 模块名称     | 介绍                                                         |
| :----------- | :----------------------------------------------------------- |
| 处理器核仿真 | 主要是仿真外设的指令集，中断等。目前可以仿真六个体系结构：arm, mips ,powerpc, blackfin, coldfire, sparc |
| 外设仿真     | 如网卡，LCD, Flash 等外设控制器的仿真模块                    |
| 统计分析     | 有代码覆盖率分析模块，函数流跟踪模块等                       |

SkyEye-1.3.6架构主要着眼于模块化和可扩展性，描述如下：

> **模块化**

每一个模块能够以 so 或者 DLL 的形式存在，并且可以独立开发，独立编译。 在SkyEye-1.3.6启动的时候会被其的核心库进行动态加载。

模块之间不存在任何依赖关系，相互独立，不存在相互调用关系。所有的模块都是调用SkyEye-1.3.6核心库提供的标准接口进行操作。

> **可扩展性**

设计一组核心的 API 及其实现。把核心的 API 和实现放在SkyEye-1.3.5的核心库中，并提供给其他外围模块。核心模块负责查找系统中可得到的其他外围模块，并进行动态加载。





## 5.重要数据结构


### 1.文本配置-Struct

SkyEye-1.3.6使用了一个文本配置文件来描述仿真的目标平台和一些其他的特性。关于SkyEye-1.3.5支持的各种配置选项，可以参考`guide/v1.3.4/`下的用户手册。我们也可以对配置文件进行扩展，来添加自己的选项。

我们用一个数据结构来代表配置文件中的一个选项：

```C
typedef struct skyeye_option_s
{
	char *option_name; /* 用来标志这个选项的字符串。 */
	int (*do_option) (struct skyeye_option_s * this_opion, 
                      int num_params, 
                      const char *params[]);/* 用来解析这个选项的函数。 */
	char* helper; /* 对选项进行描述。 */
	struct skyeye_option_t *next; /* 指向下一个 option 元素的指针。 */
} skyeye_option_t;
```

我们用一个链表来把所有的 option 管理起来。链表的头为`skyeye_option_list`变量，定义在文件`common/conf_parser/skyeye_options.c `下，代码如下：

```C
static skyeye_option_t* skyeye_option_list;
```

在`common/conf_parser/skyeye_options.c`文件中，我们还实现了对配置选项的链表进行其他的一些操作。`register_option`函数用来在表里面添加一个配置链选项。

### 2.命令行-Struct

SkyEye-1.3.6 命令行接口中的每一条命令我们都用了一个 COMMAND 的数据结构进行描述，其代码如下：

```c
struct command_s
{
	char *name; /* 敲入命令的字符串 */
	rl_icpfunc_t *func; /* 执行命令的函数.  */
	char *doc; /* 这条命令的帮助信息。 */
	struct command_s *next; /* 指向下一条命令数据结构的指针。 */
};
typedef struct command_s COMMAND;
```



我们把所有命令的数据结构用一个链表管理起来。 

```C
static COMMAND *command_list;
```





### 3.正在更新



## 6.启动流程

整个流程可以概括为：

`设置运行环境` ->  `加载动态模块及初始化系统` -> `加载命令行接口` -> `等待命令输入` -> `运行`




### 设置运行环境



可以是 elf镜像文件的名称，要运行机器的大小端等信息。在这里，我们的这些运行环境变量主要是通过解析 skyeye 的命令行参数和镜像文件获得。




### 动态模块加载

SkyEye-1.3.5启动的时候会调用模块加载函数，从系统安装的默认目录中，查找符合规范的动态链接库。实现代码位于 `common/module/skyeye_module.c`当中。

如果你在命令行模式下启动SkyEye-1.3.5，并输入`list-modules`，将会得到当前所有动态库的简要信息。主要动态模块有：

| Module Name |    File Name    |
| :---------: | :-------------: |
|    sparc    |   libsparc.so   |
|    mips     |   libmips.so    |
|    flash    |   libflash.so   |
|  code_cov   |  libcodecov.so  |
|    uart     |   libuart.so    |
|  gdbserver  | libgdbserver.so |
|  coldfire   | libcoldfire.so  |
|     arm     |    libarm.so    |
| touchscreen |    libts.so     |
|     net     |    libnet.so    |
|  nandflash  | libnandflash.so |
|     ppc     |    libppc.so    |
|    bfin     |   libbfin.so    |

每个模块需要实现两个函数`module_init`和`module_fini`，其中`module_init`函数会在动态模块加载的时候被自动执行，而 `module_fini`函数会在模块卸载的时候被动态自动调用。 

为了区分SkyEye-1.3.6 的动态模块和其他动态链接库（比如第三方组件的），每个动态模块需要定义一个全局的字符串变量, `skyeye_module`, SkyEye-1.3.6 会通过判断当前的动态链接库中是否存在变量`skyeye_module`来决定这个动态链接库是否是合法的。




### 加载命令行接口

命令行被载入以后，可以输入命令指示SkyEye的下一步动作了。一般输入命令为：`start`完成仿真环境的加载，`run`开始运行，`stop`暂停运行，`quit`退出仿真。同时你可以输入`help`了解其他的命令。
