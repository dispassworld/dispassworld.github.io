---
title: blog规划
date: 2026-07-04 13:52:10
tags:
 - 技术分享
 - 学习笔记
---

接下来会blog预期将首先记录如下一些内容
tutorials

1. Ai算法
    1. 数据基础
        1. 矩阵运算
        2. 三角函数
        3. 傅立叶级数
        4. 导数与链式法则
        5. logit和softmax/sigmoid
        6. 矩阵的秩与逆矩阵
    2. 机器学习
        1. FNN结构
        2. CNN结构
        3. RNN结构
        4. Transformer结构
            1. scaled dot-multiply attention
            2. multi-head attention
            3. 位置编码
        5. llm结构
        6. 并行化
            1. dp
            2. tp
            3. pp
        7. 量化
        8. 量化
2. Ai工程
    1. 推理引擎
        1. vllm
            1. 系统结构
            2. 一次推理的完成调用流程
            3. 系统线程模型
            4. 线程模型
            5. KV cache
            6. PagedAttention
            7. prefix cache
            8. continuous batch schedule
    2. cuda
        1. Gpu硬件结构
            1. Cpu和gpu和系统内存的物理拓扑关系
            2. 计算
                1. SM
                2. warp和warp scheduler
            3. 存储
                1. HBM
                    1. x0 GB的HBM， 2TB/s
                    2. 32B为一个最小访问单元，如果一个warp内32线程连续访问128B数据且地址对齐可以一次访问完成
                2. Shared Memory
                    1. x0 MB的SRAM， 20TB/s
                    2. 组织为32个bank的形式，如果一个warp内32线程访问不同bank则可以并行访问32个bank，否则只能分多次对相同bank发起请求（类似行选信号之后再列选）
                3. register
                4. L1/L2 cache
            4. 通信
        2. Cuda软件编程模型
            1. 存储模型
                1. 放在寄存器的变量
                2. 放在sram的变量
                3. 放在hbm的变量
            2. 执行模型
                1. SM调度多个block，根据sram和register使用量确定可调度的block数量
                2. SM内多个warp，根据执行状态切换block内不同warp执行掩盖读写延迟
            3. 线程同步
                1. warp内threads间同步阻塞等待执行__syncwarp()
                2. block内threads间同步阻塞等待执行__syncthreads()
                3. hbm的内存屏障__threadfence()
                4. 原子操作
            4. 通信模型
                1. 实际上就是这几种行为
                    1. 广播数据
                    2. 分发数据
                    3. 收集数据
                    4. 聚合（sum,max等）数据
                2. 12N
                    1. Broadcast
                    2. Scatter
                3. N21
                    1. Reduce
                    2. Gather
                4. N2N
                    1. All-Reduce
                    2. All-Gather
                    3. Reduce-Scatter
                    4. All-All
        3. 算子优化
            1. GEMM矩阵乘法-block形状与内存访问优化
            2. flashattention
                1. V1
                    1. online softmax-m,l中间变量
                    2. 分块大小Kc=M/4d,Kr=min(Kc, M/4d)
                    3. HBM访问次数优化
                        1. 2Nd+N/Kc*Nd*2= 2Nd+4d^2*N^2/M
                        2. 原始为3Nd+3*N*2
                    4. SRAM内存占用优化
                        1. Q,K,V,O,m,l
                        2. Kc*d,Kr*d,Kc*d,Kc*Kr, Kc * 2
                2. V2
                    1. Softmax归一化放到最后整好O计算完毕后
                    2. Q作为外循环优化并行度为batchSize-head numer-Q block number
                3. v3
                    1. 利用硬件独立的io组件TMA，结合ping-pong读写分离，把存算并行
                    2. 利用WGMMA硬件把softmax和gemm计算并行
                    3. 
3. 操作系统基础
    1. 计算资源管理-进程管理
    2. 存储资源管理-内存管理
        1. 物理内存与虚拟内存
        2. 虚拟内存管理-虚拟地址空间的申请与释放
            1. 红黑树管理已经申请的空间做缺页快速查找
            2. 双向链表用于遍历和范围查找，相邻区域合并
        3. 物理内存管理-物理内存的申请与释放（buddy算法）
            1. 每个内存分区维护free_area [MAX_ORDER] 数组
            2. 申请时查找最适合尺寸并拆分
            3. 释放时检查同伴情况并递归向上合并
        4. 内核业务对物理内存分配的优化-slab
            1. 根据业务需要，创建不同尺寸的kmem_cache
            2. 物理页内部分成多个小区域
        5. Malloc的用户态内存管理
            1. 桶数组管理的内存池（per-thread加速）
                1. 保留固定大小的区块不合并快速申请释放
                2. 
            2. Brk拓展堆上限
            3. Mmap直接分配大块内存区域
    3. 存储资源管理-磁盘管理
        1. 磁盘逻辑结构
        2. Inode与文件系统
    4. IO模型
        1. 同步阻塞
        2. 同步非阻塞
        3. 异步IO
        4. IO复用
            1. select/poll
            2. epoll
    5. 线程同步与互斥
        1. race condition现象
        2. 原子操作
        3. 互斥量
4. c++语言
    1. 右值与左值
    2. 移动与拷贝
    3. 完美转发
    4. move和forward原理
        1. move
        2. Forward
    5. 并发编程与原子变量
        1. Cpu硬件的逻辑模型
            1. cache与理论上的MESI协议
            2. store buffer
            3. invalidate queue
            4. x86的TSO内存一致性模型与arm的松散内存模型所允许的指令乱序
                1. StoreStore
                2. LoadLoad
                3. StoreLoad
                4. LoadStore
        2. 原子性
        3. 可见性
            1. store buffer
            2. invalid queue
        4. 有序性
            1. 编译器指令重排
            2. store buffer
            3. invalidate queue
        5. c++提供的原子序
            1. acquire-release提供的happens-before有序
            2. seq_cst 全局有序
            3. relaxed
    6. 线程互斥与同步
        1. 原子操作
        2. 自旋锁
        3. 互斥量
            1. 自旋锁
            2. 调度器到睡眠队列
        4. 信号量
            1. 没有所有权，任何线程都可以+-信号量
            2. 可以用于同步而不仅是互斥
            3. 只便于解决单一变量类型的条件，复杂多变量条件困难
        5. 条件变量
            1. 内核态实现原因
            2. spurious wakeup虚假唤醒
            3. 惊群效应
            4. 可以用来解决多变量复杂条件场景
        6. 生产者消费者经典问题
            1. 互斥量版本
                1. 1个互斥量+1个队列长度计数器
            2. 俩（多生产者消费者）信号量+一互斥量版本
                1. 1个信号量记录是否队列满
                2. 1个信号量记录是否队列空
                3. 1个互斥量保护队列的并发访问
            3. 俩（多生产者消费者）条件变量+一个队列长度计数器+一互斥量版本
                1. 1个条件变量记录是否队列满
                2. 1个条件变量记录是否队列空
                3. 1个队列长度计数器用户条件变量while检查避免spurious wakeup
                4. 一个互斥量保护队列的并发访问
        7. 多线程顺序输出123经典问题
            1. 3个信号量版本
                1. 信号量1初始1
                2. 信号量2初始0
                3. 信号量3初始0
            2. 3个（不怕惊群性能问题也可以只用1个）条件变量版本+1个节拍计数器
                1. 和信号量类似，每个条件变量表示当前节拍是否为1/2/3
                2. 每个线程有自己的条件变量睡眠等待不会惊群
    7. 无锁队列
        1. ringbuffer数组版本
        2. 链表版本
            1. enqueue
            2. dequeue
            3. aba问题
                1. hazard pointer
                2. 内存池
                3. stamped pointer
    8. 协程
        1. 与线程区别
        2. 有栈协程和无栈协程
    9. 智能指针
        1. shared_ptr
            1. T *
            2. Control block *
                1. Atomic<Strong cnt>
                2. Atomic<Weak cnt>
            3. 控制块操作线程安全，数据指针操作线程不安全
        2. weak_ptr
        3. unique_ptr
5. Ros
    1. Topic/service/action
    2. 线程模型
6. ZeroMQ
    1. 
