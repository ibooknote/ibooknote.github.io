# MMIXware A RISC Computer for the Third Millennium 阅读笔记

## 1、Readme

2020年03月24日 

ref: *MMIXware A RISC Computer for the Third Millennium* (Pv). 

### 1.1、本书涵盖的程序集合

* MMIXAL(an assembler)-- converts MMIX symbolic files to MMIX object files.
* Two simulators-- execute programs in given object files.
	* MMIX-SIM/simply MMIX: 每次执行一条指令，并允许调试
	* MMMIX: 模拟高性能管道，各部分计算在其中可同时进行

本书对应的程序只提供简单的终端界面，用于用户输入命令，计算器输出结果，有兴趣的读者可设计一个更友好的GUI程序。

### 1.2、CWEB

书中程序使用 CWEB 编写，CWEB--将 C和 TEX 组合在一起，标准的预处理器可将 mmixal.w转换成可编译的 mmixal.c 或者文档文件 mmixal.tex。

关于 CWEB 的完整信息，可参考 《The Stanford GraphBase》书中的 “How to read CWEB programs“ P70-73。

对MMIX真正感兴趣的读者，在阅读本书之前，应该先阅读第一本fascicle（分册）《The Art of Computer Programming Volume 1, Fascicle 1 MMIX》对 MMIX 和 MMIX汇编语言的介绍。

CWEB资源：

* 网址：https://www-cs-faculty.stanford.edu/~knuth/cweb.html
* 程序下载FTP服务器：ftp://ftp.cs.stanford.edu/pub/cweb/
* The CWEB System of Structured Documentation(cwebman.pdf)
* 


### 1.3、程序清单

* 1）、MMIX 解释 MMIX架构
* 2）、MMIX-ARITH 浮点算术运算子程序
* 3）、MMIX-CONFIG 处理 MMMIX 配置文件
* 4）、MMIX-IO 输入/输出子程序
* 5）、MMIX-MEM 处理特定情况下 MMMIX 与输入/输出的内存映射
* 6）、MMIX-SIM 非管道式模拟器
* 7）、MMIXAL 汇编语言
* 8）、MMMIX meta-simulator驱动程序
* 9）、MMOTYPE 用于将 MMIX对象 转换成人可读形式的工具程序

MMIX虽然格式化为 CWEB 文档，但不是真正意义上的程序，而是MMIX的完整定义，这一节应该最先阅读，其他程序可按任意顺序，但 MMIXAL 或 MMIX-SIM 应该在MMIX后阅读，而MMIX-PIPE放到最后。








