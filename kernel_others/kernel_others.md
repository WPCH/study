[TOC]

## 时间
* clocksoure
* clockevent
* tick
```
Period/Oneshot
Hrtimer
hz
Hz-->nohz-lr
|-->nohz-hr   tick中处理hrtimer(hrtimer_run_queues)并在timer_softirq里判断和切换至nohz-lr or nohz-hr
 tick用hrtimer实现（nohz-hr）

idle 中的处理
  设置下一个tick尽量晚到期，max_idle_ns
  若关闭local timer，设置Broadcast tick
```
* tick_sched
```
tick_sched_do_timer->update_jiffies64->update_wall_time->timekeeping
update_process_times->account_process_tick(user/sys(irq,sirq,sys)/idle)
                      run_local_timers->raise TIMER_SOFTIRQ
                      rcu_check_callbacks
                      scheduler_tick
                      run_posix_cpu_timers
```

## SMP boot
<img src="pictures/1.png" width = "500" height = "230" align=center />

bootCPU为secondaryCPU创建idle线程，设置运行地址，唤醒secondcpu secondary_startup

## 设备模型
```
device driver bus-----------|
kobject ktype uevent--------|---->sysfs
kset------------------------|
class(往往是符号链接)--------|
```

## 电源管理
```
suspend/resume

wakeup
  wakeup_source
    pm_stay_awake(activate wakeup event)
    pm_relax(deactivate wakeup event)

  event_count、active_count、wakeup_count（阻止休眠的次数）

  外部产生，如中断
  内部产生，如防止执行某些代码时休眠
    用户空间（wake_lock）
```

## CPU core管理
* CPU idle
```
governor:
  ladder(periodic timer tick system)
  menu(tickless system)
    predicted_us  latency_req
```
* cpufreq （包含regulator）
```
frequency table
governor
```
* cpuhotplug
> cpu_up
>
><img src="pictures/2.png" width = "500" height = "250" align=center />

> cpu_down


## ARM boot
```
Asm part:
  关键行为：
  stext->__create_page_tables->__enable_mmu->__mmap_switched->start_kernel

  参数的处理：
  r0  = cp#15 control register
  r1  = machine ID
  r2  = atags/dtb pointer
  r9  = processor ID

    .long   processor_id            @ r4
    .long   __machine_arch_type     @ r5
    .long   __atags_pointer         @ r6
  #ifdef CONFIG_CPU_CP15
    .long   cr_alignment            @ r7
  #else
    .long   0               @ r7
  #endif
     .long   init_thread_union + THREAD_START_SP @ sp
     .size   __mmap_switched_data, . - __mmap_switched_data

  ARM(   ldmia   r3, {r4, r5, r6, r7, sp})
  THUMB( ldmia   r3, {r4, r5, r6, r7}    )
  THUMB( ldr sp, [r3, #16]       )
     str r9, [r4]            @ Save processor ID
     str r1, [r5]            @ Save machine type
     str r2, [r6]            @ Save atags pointer
     cmp r7, #0
     strne   r0, [r7]            @ Save control register values

  1. create page tables to enable mmu
  2. call start_kernel with param
```
