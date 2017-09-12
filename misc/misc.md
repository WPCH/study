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
