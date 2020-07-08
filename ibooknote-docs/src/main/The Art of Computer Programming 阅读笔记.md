# The Art of Computer Programming 阅读笔记

## 1、TAOCP为什么没有使用高级语言讲解算法程序示例？

2020年03月13日 

ref: *The Art of Computer Programming Volume 1, Fascicle 1 MMIX* (Piv). 

使用机器语言（Machine language）的原因：  

* TAOCP的一个重要目标是为了展示上层结构（hight-level constructions）在机器上是如何实现的，而不是简单的展示如何应用。例如：
	* coroutine linkage
	* tree structures
	* random number generation
	* high-precision arithmetic
	* radix conversion
	* packing of data
	* combinatorial searching
	* recurion

* 书中的程序通常都比较简短，很容易抓住重点。  
* 如果想更深入的理解计算机，至少应该知道硬件底层是什么样的，否则写出来的程序可能会很古怪。  
* 机器语言在很多情况下都是必不可少的。
* 通过使用机器语言描述排序、搜索算法的基本方法，能够更好的理解缓存和RAM大小，以及其他硬件特性（内存速度、管道、多核、备用缓冲区、缓存块大小等）对算法的影响。

另外，如果用一种高级语言来编写书中的示例，那么在60年代可能得选择 Algol W，在70年代得用Pascal重写，在80年代得用C重写，90年代得用C++重写，然后用Java重写，到了2000年，可能又得用另外一种流行的语言重写。

## 2、相关基础知识

### 2.1、Bits and bytes

MMIX一次处理64位，并涉及到二进制和十六进制之间的转换，使用简洁的术语表示不同字节的大小：

* 8 bits = 1 bytes
* 2 bytes = 1 wyde
* 2 wydes = 1 tetra
* 2 tetras = 1 octa

使用最左边的位表示Integers的符号（sign），需理解负数的计算方法（通过补码计算）。

## 2.2、内存和寄存器

MMIX计算机架构：

* 2^64个内存单元 M[0]-M[2^64-1]
* 256个通用寄存器 $0-$255
* 32个特定寄存器 rA-rZ, rBB, rTT, rWW, rXX, rYY, rZZ

## 2.3、指令（Instructions）

指令格式：

> OP, X, Y, Z

含义：

* OP 操作码（operation code/opcode）
* X, Y, and Z 操作数（operands）

