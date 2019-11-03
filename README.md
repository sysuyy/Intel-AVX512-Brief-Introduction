# Intel AVX-512 Brief Introduction
Intel AVX-512简介

<br />

## 15.1 概述

Intel AVX-512家族由一组指令集扩展集构成，包括了AVX-512基础、AVX-512指数与倒数指令、AVX-512冲突、AVX-512预取，以及额外的512位SIMD指令扩展。intel AVX-512指令是对AVX与AVX2的自然扩展。Intel AVX-512引入了下列架构上的提升：

• 支持512位宽度的向量与SIMD寄存器组。512位寄存器状态受操作系统管理，通过使用XSAVE/XRSTOR指令，这对指令在45nm的Intel 64处理器中就已引入。

• 在64位模式下，支持16个新增的512位SIMD寄存器（总共具有32个额SIMD寄存器，从ZMM0到ZMM31）。这新增的16个寄存器状态同样受操作系统使用XSAVE/XRSTOR/XSAVEOPT指令进行管理。

• 支持8个屏蔽操作（**opmask**）寄存器（从k0到k7），用于带条件的执行以及高效的目的操作数的合并。操作屏蔽寄存器的状态同样受操作系统使用XSAVE/XRSTOR/XSAVEOPT指令进行管理。

• 添加了一个新的编码前缀（称作为 **EVEX**）以支持将向量长度编码增加到512位。EVEX前缀基于VEX前缀作为基础进行构建，提供了紧凑的、高效的可用于VEX编码的功能性，外加下列增强的向量特性：

    • 操作屏蔽。
    • 内嵌的广播。
    • 内嵌的指令前缀舍入控制。
    • 压缩的地址位移
    
<br />

#### 15.1.1 512位宽度的SIMD寄存器支持

Intel AVX-512指令支持512位宽度的SIMD寄存器（ZMM0-ZMM31）。ZMM寄存器的低256位与相应的YMM寄存器对应，而低128位则与相应的XMM寄存器对应。

<br />

#### 15.1.2 32个SIMD寄存器支持

Intel AVX-512指令也支持64位模式下32个SIMD寄存器（XMM0-XMM31，YMM0-YMM31，以及ZMM0-ZMM31）。在32位模式下，可用的向量寄存器的个数依旧是8。

<br />

#### 15.1.3 八个屏蔽操作寄存器支持

AVX-512指令支持8个屏蔽操作寄存器（k0-k7）。每个屏蔽操作寄存器的宽度在架构上定义为MAX_KL的大小（最大64位）。八个屏蔽操作寄存器的其中七个（k0-k7）可以被用于跟EVEX编码的AVX-512基础指令相结合以提供带条件的执行以及目的操作数中数据元素的高效合并。屏蔽操作寄存器k0的编码一般是在所有数据元素都需要的情况下（即无条件处理）进行使用。此外，屏蔽操作寄存器也被用于向量标志/元素级的向量源以引入新奇的SIMD功能性，比如可以参考**VCOMPRESSPS**。

![avx512-register-scheme](https://github.com/zenny-chen/Intel-AVX512-Brief-Introduction/blob/master/avx512-register-scheme.png)

<br />

#### 15.1.4 指令语法增强

EVEX编码的架构以以下方式增强了向量指令编码模式：

• 512位向量长度，多达32个ZMM寄存器，使用增强的VEX（EVEX）支持向量编程环境。

EVEX前缀提供了比起VEX前缀更多的可编码的位域。除了在64位模式下对32个ZMM寄存器编码之外，使用EVEX前缀编码的指令可以直接对屏蔽操作寄存器操作数（一共8个）的其中7个进行编码，以提供向量指令编程的带条件处理。增强的向量编程环境可以显式地用包含以下元素的指令语法进行表达：

• 一个屏蔽操作操作数：屏蔽操作寄存器使用“k1”到“k7”的记号进行表达。一个EVEX编码的指令支持使用屏蔽操作寄存器 k1 的带条件的向量操作，这可以表示为在目的操作数之后使用记号 **{k1}** 来表示。对这条特征的使用对于大部分指令都是可选的。有两种类型的屏蔽方式（合并与清零），通过使用 **EVEX.z** 位进行区分（在指令签名中用 **{z}** 表示）。

• 内嵌的广播对于某些指令可以在源操作数上支持，该源操作数可以被编码为一个存储器向量。一个存储器向量的数据元素可以带条件的去获取或写入。

• 对于仅操作在SIMD寄存器中带有舍入语义的浮点数据上的指令语法，EVEX编码可以显式地在EVEX位域中提供舍入控制，或以标量长度，或以512位向量长度。

在512位指令中，对源操作数的所有元素的向量加法可以用与AVX指令相同的语法来表达：

```asm
VADDPS    zmm1,  zmm2,  zmm3
```

此外，AVX-512基础的EVEX编码模式可以将带条件的向量加法表示为：

```asm
VADDPS    zmm1  {K1} {z},  zmm2,  zmm3
```

