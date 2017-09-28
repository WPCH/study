## CPU架构
[TOC]

### 体系结构(ISA)
```
CISC 指令数量多，功能多，变长
RISC 指令数量少，功能简单，定长
```
>![img](pictures/1.png)

### 微架构
#### pipeline
##### 经典5级流水线
>![img](pictures/2.png)

##### 流水线冒险
###### 结构冒险
>![img](pictures/3.png)

###### 数据冒险
>![img](pictures/4.png)
>
>![img](pictures/5.png)
>
>![img](pictures/6.png)
>
>![img](pictures/7.png)
>
>![img](pictures/8.png)

###### 控制冒险
```
如果程序的实际执行路径是要跳转到其他的地址去执行，那么流水线中已经做 的取指和译码工作就白做了。

处理：硬件冲刷流水线Or软件增加Nop指令排空流水线
```

##### 分支预测
```
根据历史信息来对跳转指令进行预测

典型分支预测算法：
```
```
    1位预测
```
>![img](pictures/9.png)
```
    2位预测
```
>![img](pictures/10.png)

##### 乱序执行
```
某个指令执行时需要等待，先执行后面不依赖该数据的指令
```
###### 指令相关性
>![img](pictures/11.png)

###### 去除指令相关性
* 去数据相关：软件尽量减少
* 去控制相关：分支预测，投机执行
* 去伪相关：将ISA寄存器重新映射到处理器内部的物理寄存器（寄存器重命名）
>![img](pictures/12.png)

###### 乱序执行实现
>![img](pictures/13.png)
```
指令调度
  分析指令间的相关性，分析指令什么时候能开始执行。
  指令能否开始执行， 依赖于两个条件：
  （1）是否有空闲的功能单元去执行这条指令。
  （2）该指令的源操作数是否已经准备好。

顺序提交
  乱序执行后，指令的这个结果并没有立即提交到ISA 寄存器中，而是先缓存起来，只有当前指令前面的指令提交后，这条指令才能提交。
```
>![img](pictures/14.png)

##### 并行
###### 指令并行
* Superscaler 处理器实现，兼容
>![img](pictures/15.png)
* VLIW 软件实现
>![img](pictures/16.png)
* Superscaler实例 P4流水线
```
  前端
```
>![img](pictures/17.png)
```
  后端
```
>![img](pictures/18.png)

###### 数据并行
```
SIMD
  ARM NEON
```

###### 线程并行
* 硬件多线程
```
  粗粒度
```
>![img](pictures/19.png)
```
  细粒度
```
>![img](pictures/20.png)
```
  同时多线程（SMT）
```
>![img](pictures/21.png)
* 多核

#### Cache-memory系统
><img src="pictures/22.png" width = "600" height = "350" align=center />
##### 映射方式
```
1. Full- associative Cache（全关联Cache） 任意映射
2. Direct- mapped Cache（直接映射Cache）
3. Set- associative Cache（组关联Cache）
```
>![img](pictures/23.png)![img](pictures/24.png)![img](pictures/25.png)

##### Cache write
```
write through(write buffer)
write back
  Cache miss
    no-write-allocate
    write allocate
      Random FIFO LRU
```
##### Cache地址
```
1. VIVT（Virtual index Virtual tag）。寻找cache set的index和匹配cache line的tag都是使用虚拟地址。存在cache alias
2. PIPT（Physical index Physical tag）。寻找cache set的index和匹配cache line的tag都是使用物理地址。
3. VIPT（Virtual index Physical tag）。寻找cache set的index使用虚拟地址，而匹配cache line的tag使用的是物理地址。Set index+offset>12 存在cache alias
```
##### Cache一致性
###### MESI protocol
>![img](pictures/26.png)
>
>![img](pictures/27.png)

###### Cache系统结构（MESI之类协议引起，带来memory order问题，引入memory barrier来解决）
>![img](pictures/28.png)
