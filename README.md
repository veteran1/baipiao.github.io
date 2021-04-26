# 8051-ELL文档中心

<img src="_media/download.svg" alt="logo" style="zoom:10%;" /> <font color=#0ddff>[ <u>**从gitee下载8051-ELL**</u>](https://gitee.com/zeweni/ELL-8051-LIB) </font> 

[![](https://img.shields.io/badge/version-1.1.6-green)](https://github.com/zewen-i/8051-ELL-LIB) [![](https://badgen.net/github/license/zewen-i/8051-ELL-LIB?color=orange)](https://github.com/zewen-i/8051-ELL-LIB) [![](https://badgen.net/github/stars/zewen-i/8051-ELL-LIB?color=green)](https://github.com/zewen-i/8051-ELL-LIB) [![](https://badgen.net/github/forks/zewen-i/8051-ELL-LIB)](https://github.com/zewen-i/8051-ELL-LIB) <a href='https://gitee.com/zeweni/ELL-8051-LIB/stargazers'><img src='https://gitee.com/zeweni/ELL-8051-LIB/badge/star.svg?theme=white' alt='star'></img></a> <a href='https://gitee.com/zeweni/ELL-8051-LIB/members'><img src='https://gitee.com/zeweni/ELL-8051-LIB/badge/fork.svg?theme=gray' alt='fork'></img></a>



8051-ELL库，是根据新一代增强型8051为内核的MCU，基于keil开发的软件包。函数库采用了LL库的编程思想，充分考虑8051的特性，结合硬件条件，提供大量标准的API函数，供开发者访问底层硬件细节。并且函数库的大小可裁剪，在代码密度和执行效率上做了很好的平衡。

函数库遵循 Apache 许可证 2.0 版本，可免费在商业产品中使用，不需要公布应用程序源码，没有潜在商业风险。

<br>

## 支持的型号及外设

ELL库率先支持了STC8系列MCU，部分型号可能有所差异，详情可查看官方数据手册。

> `√`代表已经支持、`空`代表MCU没有这个外设、 ` X`代表还没有适配


| 型号      | 定时器 | IO   | 中断 | 系统时钟 | PCA  | PWM  | MPWM | HPWM | EEPROM | ADC  | MDU16 | 比较器 | USB  | LED  | RTC  | TKEY |
| --------- | ------ | ---- | ---- | -------- | ---- | ---- | ---- | ---- | ------ | ---- | ----- | ------ | ---- | ---- | ---- | ---- |
| STC8A系列 | √      | √    | √    | √        | √    | √    | X    | X    | √      | √    |       | √      |      |      |      |      |
| STC8C系列 | √      | √    | √    | √        |      |      |      |      | √      |      | √     | √      |      |      |      |      |
| STC8F系列 | √      | √    | √    | √        |      |      |      |      | √      |      |       | √      |      |      |      |      |
| STC8G系列 | √      | √    | √    | √        | √    |      | X    |      | √      | √    | √     | √      | X    | X    |      |      |
| STC8H系列 | √      | √    | √    | √        |      |      |      | X    | √      | √    | √     | √      | X    | X    | X    | X    |



