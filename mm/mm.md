## 内存管理

* [初始化](#初始化)
* [页表（ARM）](#页表arm)
* [bootmem](#bootmem)
* [memblock](#memblock)
* [ZONE](#zone)
* [内存分配](#内存分配)
* [进程地址空间](#进程地址空间)
* [内存回收](#内存回收)
* [内存规整](#内存规整)
* [高端内存相关(虚拟地址空间 ARM32)](#高端内存相关虚拟地址空间-arm32)
* [page cache](#page-cache)

### 初始化

> zone的初始化
>> start_kernel->setup_arch->paging_init->bootmem_init->zone_size_init->free_area_init_node->free_area_init_core->memmap_init_zone

> 页面从bootmem释放到buddy系统
>> start_kernel->mm_init->mem_init->free_all_bootmem->free_low_memory_core_early->__free_memory_core->_free_page_memory->__free_pages_bootmem->__free_pages->__free_pages_ok->__free_one_page

### 页表（ARM）
```
mmu pgd 4096个，kernel pgd 2048个（bit[31：21]）,两个合成一个，指向两个连续的pte(512个entry)
      pgd             pte
  |        |
  +--------+
  |        |     +------------+ +0
  +- - - - +     | Linux pt 0 |
  |        |     +------------+ +1024
  +--------+ +0  | Linux pt 1 |
  |        |---->+------------+ +2048
  +- - - - + +4  |  h/w pt 0  |
  |        |---->+------------+ +3072
  +--------+ +8  |  h/w pt 1  |
  |        |     +------------+ +4096
```

### bootmem
```
以位图代替原来的空闲链表结构来表示存储空间
```

### memblock
```
memblock.memory  memblock.reserved
```

### ZONE
```c
struct zone {
     ...
     unsigned long watermark[NR_WMARK];//min low high
     ...
     long lowmem_reserve[MAX_NR_ZONES];//zone中保留的页框数
     ...
     struct per_cpu_pageset __percpu *pageset;//percpu页框高速缓存
     ...
     struct free_area    free_area[MAX_ORDER];//空闲页框块
     ...
     spinlock_t      lock;
     ...
     spinlock_t      lru_lock;
     struct lruvec       lruvec;
}

struct free_area {
    struct list_head    free_list[MIGRATE_TYPES];
    unsigned long       nr_free;
};
```

### 内存分配
[内存分配](./内存分配/内存分配.md)
### 进程地址空间
[进程地址空间](./进程地址空间/进程地址空间.md)
### 内存回收
[内存回收](./内存回收/内存回收.md)

### 内存规整
[内存规整](http://www.cnblogs.com/tolimit/p/5286663.html)

### 高端内存相关(虚拟地址空间 ARM32)
```
pkmap  kmap永久映射(阻塞) 内核指定了需要进行映射的页面，若不属于高端内存则返回线性地址
vmalloc
fixmap  FIX_KMAP --- kmap_atomic临时映射(不阻塞，覆盖) 内核指定...同kmap
```

### page cache
```
1. 普通文件的page cache
2. block device 的page cache
3. swap cache
```
