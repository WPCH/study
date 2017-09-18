[TOC]

### glibc ptmalloc
```
主分配区: heap mmap
非主分配区 ：mmap
```
#### chunk
> in use
>
> <img src="pictures/1.png" width = "300" height = "100" align=center />
>
> free
>
> <img src="pictures/2.png" width = "360" height = "220" align=center />

#### bins(空闲chunk容器)
> <img src="pictures/3.png" width = "400" height = "180" align=center />
```
small bins中的chunk 按照最近使用顺序进行排列
large bins 中的每一个 bin 分别包含了一个给定范围内的 chunk，其中的 chunk 按大小序排列。相同大小的 chunk 同样按照最近使用顺序排列。
fast bins: 不大于 max_fast（默认值为64B）的chunk 被释放后，首先会被放到 fast bins中
unsorted bin 的队列使用 bins 数组的第一个， 如果被用户释放的 chunk 大于 max_fast，或者 fast bins 中的空闲 chunk 合并后，这些 chunk 首先会被放到 unsorted bin 队列中
top chunk:
  非主分配区 sub-heap top
  主分配区的 top chunk 在第一次调用 malloc 时会分配一块(chunk_size + 128KB)align 4KB 大小的空间作为初始的 heap
mmaped chunk
last remainder

分配顺序：
fast bins-->(unsorted bin)bins-->top chunk-->mmaped chunk
small chunk：small bins-->last remainder
```

### RT
* 临界区可抢占（部分spinlock换成mutex，按优先级排队）
>优先级反转
>
><img src="pictures/4.png" width = "400" height = "180" align=center />
* 高精度时钟
* 中断线程化 (可选，如tick没有线程化)
* 软中断线程化 (同上)
* 实时调度算法

### 栈溢出调试
><img src="pictures/5.png" width = "500" height = "280" align=center />
>
>3. gcc  -fstack-protector

### 红黑树
#### 二叉树
##### 结构
><img src="pictures/14.png" width = "250" height = "130" align=center /><img src="pictures/15.png" width = "260" height = "130" align=center /><img src="pictures/16.png" width = "330" height = "180" align=center />

#### 2-3查找树
##### 结构
><img src="pictures/17.png" width = "320" height = "150" align=center />
##### 构造过程
><img src="pictures/18.png" width = "450" height = "700" align=center />

#### 红黑树
##### 结构
><img src="pictures/19.png" width = "320" height = "320" align=center />
##### 构造过程
><img src="pictures/20.png" width = "600" height = "700" align=center />

### 其他
```
编译、链接原理，链接脚本编写
  C==>asm，数据段中数据地址保存在text段中(Label)，用于访问
```
#### 数据长度
><img src="pictures/6.png" width = "300" height = "180" align=center />

#### Debug
```
Objdump、addr2line
Ftrace
gdb
Strace、ltrace
perf
```

#### Uboot
```
Relocation
  指令：b,bl地址无关
  全局变量：利用rel.dyn段修改地址
            rel.dyn段存有label地址，加上offset找到新地址，对其内容加上offset即为全局变量新地址
Sdram空间
  top-->hide mem-->logbuff-->pram-->tlb space(16K)-->framebuffer space-->uboot code space-->addr-->malloc len-->bd len-->gd len-->fdt-->12 byte-->addr_sp
```

#### 开源许可证
><img src="pictures/21.png" width = "620" height = "350" align=center />

#### driver
##### USB
>![img](pictures/7.png)
>
><img src="pictures/8.png" width = "350" height = "180" align=center />
>
><img src="pictures/9.png" width = "120" height = "180" align=center /> <img src="pictures/10.png" width = "300" height = "180" align=center /> <img src="pictures/11.png" width = "250" height = "180" align=center />
* Setup:
> 标准设备请求
>
> <img src="pictures/12.png" width = "450" height = "300" align=center />
> <img src="pictures/13.png" width = "450" height = "700" align=center />

##### UBI
###### UBI简介
>**Overview**
>
>UBI全称"Unsorted Block Images"。它是工作于raw flash devices之上的volume管理系统，它管理一个单一physical flash设备上的多个logical volume，能够把I/O负载均匀的分发到flash chip上。
>
>在一定意义上，UBI可以和Logical Volume Manager相比较。LVM映射logical sector到物理sectors，UBI映射logical eraseblcok到physical eraseblocks。但是除了映射，UBI还实现了wear-leveling和透明的I/O错误的管理。
>
>一个UBI volume是一组连续的logical eraseblocks(LEBs)。每一个逻辑eraseblcok可以被映射为任意的physical eraseblock。这个映射是由UBI管理，并且对上层隐藏了global wear-leveling机制（记录per-physical eraseblock erase counters 以及透明的移动more worn-out数据到less worn-out上）。
>
>UBI volume在创建时确定尺寸大小，也可以在日后改变（volume是动态re-sizable）。UBI有user-space工具可以用来管理UBI volumes。
>
>有两种UBI volume，动态的和静态的。静态volumes是只读的，内容由CRC-32 checksums保护；而动态volume是read-write，上层负责确保数据的完整性。
>
>UBI处理坏块，上层不需要关心坏块管理。UBI有一个保留的physical eraseblocks pool，当一个physical eraseblock变成坏块，UBI透明的用一个好physical block替换这个坏块。UBI把新出现的physical eraseblock的数据移动到好physical eraseblock。UBI volume对发生的事毫无察觉。
>
>NAND flashes在读写操作时可能会发生bit-flips。Bit-flips通过ECC checksums纠正，但是积累到一定数据可能会发生数据丢失。UBI会移动发生bit-flips的数据到另外一个physical eraseblocks。这个过程叫scrubbing。scrubbing过程是后台的，并且对上层透明。

详见[UBI介绍](http://blog.csdn.net/kickxxx/article/details/6707589#t8)

###### UBI Volume
> **Calculations**
> Usable Size Calculation
>
>As documented here, UBI reserves a certain amount of space for management and bad PEB handling operations. Specifically:
>
>    2 PEBs are used to store the UBI volume table
    1 PEB is reserved for wear-leveling purposes;
    1 PEB is reserved for the atomic LEB change operation;
    a % of PEBs is reserved for handling bad EBs. The default for NAND is 1%
    UBI stores the erase counter (EC) and volume ID (VID) headers at the beginning of each PEB. 1 min I/O unit is required for each of these.
>
> To calculate the full overhead, we need the following values:
 Symbol    Meaning     Value for XO test case
 W     total number of physical eraseblocks on the flash chip (NB: the entire chip, not the MTD partition)
 SP    PEB Size    128KiB
 SL    LEB Size    128KiB - 2 * 2KiB = 124 KiB
 P     Ttotal number of physical eraseblocks on the MTD partition  200MiB / 128KiB = 1600
 BB    number of bad blocks on the MTD partition;
 BR    number of PEBs reserved for bad PEB handling. it is 20 * W/1024 for NAND by default, and 0 for NOR and other flash types which do not have bad PEBs
 B     B - MAX(BR,BB);
 O     The overhead related to storing EC and VID headers in bytes, i.e. O = SP - SL   4KiB
>
> UBI Overhead = (B + 4) * SP + O * (P - B - 4)
       = (16 + 4) * 128Kib + 4 KiB * (1600 - 16 - 4)
       = 8880 KiB
       = 69.375 PEBs (round to 69)
