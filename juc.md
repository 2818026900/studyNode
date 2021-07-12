# JUC

## 进程与线程

### 概念

#### 进程与线程

**进程**

![image-20210709162211564](juc.assets/image-20210709162211564.png)	

**线程**

![image-20210709162235334](juc.assets/image-20210709162235334.png)	

**对比**

- 进程基本上相互独立，线程存在于进程之内，是进程的一个子集
- 进程拥有共享资源，如内存空间等，供其内部的线程共享
- 进程通信较为复杂
  - 同一台计算机的进程通信称为IPC（inter-process communication）
  - 不同计算机之间的进程通信需要通过网络，并遵循共同的协议，如HTTP
- 线程通信相对简单，他们共享进程内的内存，多个线程可以访问同一个共享变量
- 线程更轻量，线程上下文切换成本一般上比进程上下文切换成本低。

#### 并行与并发

**并发**

单核 cpu 下，线程实际还是**串行执行** 的。操作系统中有一个组件叫任务调度器，将 cpu 的时间片（windows 下时间片最小约为 15 ms） 分给不同的线程使用，一般会将 **线程轮流使用 CPU ** 的做法叫 ***并发***  

![image-20210710152053963](juc.assets/image-20210710152053963.png) 

![image-20210710152159461](juc.assets/image-20210710152159461.png) 

**并行**

多核 cpu 下， 每个核都可以调度运行线程， 这时候线程可以是 ***并行***

![image-20210710152455600](juc.assets/image-20210710152455600.png) 

![image-20210710152235221](juc.assets/image-20210710152235221.png) 

#### 应用

![image-20210710152900226](juc.assets/image-20210710152900226.png) 

![image-20210710153529947](juc.assets/image-20210710153529947.png) 

![image-20210710205923042](juc.assets/image-20210710205923042.png) 

## Java线程

### 创建与运行线程

#### 方法一：直接使用 Thread

![image-20210710213026047](juc.assets/image-20210710213026047.png) 

#### 方法二：使用 Runnable 配合 Thread

把【线程】和【任务】代码分开

- Thread 代表线程
- Runnable 可运行的任务

![image-20210710221046361](juc.assets/image-20210710221046361.png) 

#### Thread 与 Runnable 的关系

- 方法1 是把线程和任务合并在一起了，方法2 是把线程和任务分开了
- 用 Runnable 更容易与线程池等高级 API 配合
- 用 Runnable 让任务脱离了 Thread 继承体系，更灵活

#### 方法三：FutureTask 配合 Thread

FutureTask 能接收 Callable 类型的参数，用来处理有返回结果的情况

![image-20210710223255034](juc.assets/image-20210710223255034.png) 

### 查看进程线程的方法

#### Windows

- 任务管理器
- tasklist 查看进程
- taskkill 杀死进程

#### Linux

- ***ps -fe***  查看所有进程
- ***ps -fT -p <PID>***  查看某个进程的所有线程
- kill 杀死进程
- top 按大写 H 切换是否显示线程
- ***top -H -p <PID>*** 查看某个进程的所有线程

#### Java

- ***jps*** 查看所有Java进程
- ***jstack <PID>*** 查看某个Java进程的所有线程状态
- ***jconsole*** 查看某个Java进程中线程的运行情况

![image-20210710232138689](juc.assets/image-20210710232138689.png) 

### *<font color="#FF5151">原理——线程运行</font>

#### 栈与栈帧

每个线程的启动后，虚拟机都会为其分配一块栈内存

- 每个栈由多个栈帧（Frame）组成， 对应着每次方法调用时所占用的内存
- 每个线程只能有一个活动栈帧，对应着当前正在执行的那个方法

**图解** 

![image-20210710234313863](juc.assets/image-20210710234313863.png) 

![image-20210710235523588](juc.assets/image-20210710235523588.png) 

![image-20210710235823406](juc.assets/image-20210710235823406.png) 

![image-20210710235932648](juc.assets/image-20210710235932648.png) 

![image-20210711000013976](juc.assets/image-20210711000013976.png) 

![image-20210711000110365](juc.assets/image-20210711000110365.png) 

![image-20210711000216380](juc.assets/image-20210711000216380.png) 

![image-20210711000323263](juc.assets/image-20210711000323263.png) 

![image-20210711000409792](juc.assets/image-20210711000409792.png) 

#### 线程上下文切换（Thread Context Switch)

**导致cpu不在执行当前线程原因**

1. 线程的cpu 时间片用完
2. 垃圾回收
3. 有更高优先级的线程需要运行
4. 线程自己调用了sleep、yield、wait、join、park、synchronized、lock等方法

当Context Switch 发生时，需要由操作系统保存当前线程的状态，并恢复另一个线程的状态，Java中对应的概念就是程序计数器。

1. 状态包括程序计数器、虚拟机栈中每个栈帧的信息、如局部变量表、操作数栈、返回地址等
2. Context Switch 频繁会影响性能

**图解**

主线程时间片耗尽

![image-20210711174513445](juc.assets/image-20210711174513445.png) 

切换线程，保存状态

![image-20210711174617668](juc.assets/image-20210711174617668.png) 

t1线程时间片耗尽，切换回main线程

![image-20210711174723420](juc.assets/image-20210711174723420.png) 

### 常见方法

![image-20210711175206050](juc.assets/image-20210711175206050.png) 

![image-20210711175537381](juc.assets/image-20210711175537381.png) 

#### start 与 run 

![image-20210711202516328](juc.assets/image-20210711202516328.png) 

#### sleep 与 yield

##### sleep

1. 调用sleep会让当前线程从 Running 进入 Time Waiting 状态

   ![image-20210711205257214](juc.assets/image-20210711205257214.png) 

   ![image-20210711205342493](juc.assets/image-20210711205342493.png) 

2. 其他线程可以使用 interrupt 方法打断正在睡眠的线程，这时 sleep 方法会抛出 InterruptedException

   ![image-20210711210345697](juc.assets/image-20210711210345697.png) 

   ![image-20210711210422355](juc.assets/image-20210711210422355.png) 

3. 睡眠结束后的线程未必会立刻得到执行

4. 建议用 TimeUnit 的 sleep 代替 Thread 的 sleep 来获取更好的可读性

   ![image-20210711211037976](juc.assets/image-20210711211037976.png) 

   **TimeUnit sleep 内部实现** 

   ![image-20210711211136793](juc.assets/image-20210711211136793.png) 

##### yield

1. 调用 yield 会让当前线程从 Running 进入 Runnable 状态，然后调度执行其他同优先级的线程，如果这时没有同优先级的线程，那么不能保证让当前线程暂停的效果
2. 具体的实现依赖于操作系统的任务调度器

##### 线程优先级

1. 线程优先级会提示（hint）调度器优先调度该线程，但它仅仅说提示，调度器可以忽略它
2. 如果 cpu 繁忙，那么优先级高的线程会获得更多的时间片，如果 cpu 空闲，优先级几乎没有作用

![image-20210711212134196](juc.assets/image-20210711212134196.png) 

![image-20210711212530259](juc.assets/image-20210711212530259.png) 

##### <font color=" #28FF28">*案例-防止CPU占用100%</font> 

![image-20210711212759531](juc.assets/image-20210711212759531.png) 

#### join

![image-20210711213107001](juc.assets/image-20210711213107001.png) 

![image-20210711213204640](juc.assets/image-20210711213204640.png) 

![image-20210711213511314](juc.assets/image-20210711213511314.png) 

##### <font color=" #28FF28">*案例-应用同步</font> 

![image-20210711213649952](juc.assets/image-20210711213649952.png) 

##### 有时效的join

当时效小于实际线程结束时间时，join按照输入时间提前结束

但时效大于实际线程结束时间时，join按照实际线程结束实际结束

#### interrupt

##### 打断sleep、wait、join线程

![image-20210711214917138](juc.assets/image-20210711214917138.png) 

![image-20210711215320272](juc.assets/image-20210711215320272.png) 

打断标记可以被用来使线程停止

##### *<font color=" FF9D6F">设计模式-两阶段终止</font> 

**Two Phase Termination** 

在一个线程 T1 中通知 T2 终止， T2 处理完自行终止

**错误思路** 

![image-20210711220934594](juc.assets/image-20210711220934594.png) 

**实现思路**

![image-20210711221657187](juc.assets/image-20210711221657187.png) 

![image-20210711222316836](juc.assets/image-20210711222316836.png) 

##### 打断patk线程

![image-20210711223321481](juc.assets/image-20210711223321481.png) 

如果打断标记为true，则park会失效，此时应该配合Interrupted方法使用而不是isInterrupted

- 使用isInterrupted ![image-20210711223529132](juc.assets/image-20210711223529132.png) 

  ![image-20210711223544091](juc.assets/image-20210711223544091.png) 

- 使用interrupted

  ![image-20210711223638766](juc.assets/image-20210711223638766.png) 

  ![image-20210711223905424](juc.assets/image-20210711223905424.png) 

### 不推荐使用方法

![image-20210711224027996](juc.assets/image-20210711224027996.png) 

### 主线程与守护线程

默认情况下，Java 进程需要等待所有线程都运行结束，才会结束。但有一种特殊的线程叫 **守护线程** 只要其他非守护线程运行结束，即使守护线程的代码没有执行完，守护线程也会强制结束。

![image-20210711224500612](juc.assets/image-20210711224500612.png) 

**守护线程常见应用**

- 垃圾回收器线程
- Tomcat 中的 Acceptor 和 Pooler 线程都是守护线程

### 线程状态

#### 五种状态

从 **操作系统** 层面描述

![image-20210711225154713](juc.assets/image-20210711225154713.png) 

![image-20210711225419032](juc.assets/image-20210711225419032.png) 

#### 六种状态

从 **Java API** 层面描述

根据 Thread State 枚举，分为六种状态

![image-20210711230334787](juc.assets/image-20210711230334787.png) 

![image-20210711230827324](juc.assets/image-20210711230827324.png) 

### 本章小结

![image-20210711233150297](juc.assets/image-20210711233150297.png) 

## 共享模型之管程

### 共享带来的问题

#### 小故事

![image-20210711233851833](juc.assets/image-20210711233851833.png) 

![image-20210711234116414](juc.assets/image-20210711234116414.png) 

![image-20210711234149753](juc.assets/image-20210711234149753.png) 

#### Java 的体现

![image-20210711234329384](juc.assets/image-20210711234329384.png) 

#### 问题分析

![image-20210711234559529](juc.assets/image-20210711234559529.png) 

![image-20210711234720490](juc.assets/image-20210711234720490.png) 

![image-20210711234824220](juc.assets/image-20210711234824220.png) 

![image-20210711235028397](juc.assets/image-20210711235028397.png) 

![image-20210711235054354](juc.assets/image-20210711235054354.png) 

![image-20210711235307535](juc.assets/image-20210711235307535.png) 

#### 临界区 Critical Section

- 一个程序运行多个线程本身没有问题
- 问题出在多个线程访问共享资源
  - 多个线程读共享资源也没有问题
  - 在多个线程对共享资源读写操作时发生指令交错，就会出现问题
- 一段代码块内如果存在对共享资源的多线程读写操作，称这段代码块为 **临界区** 

![image-20210711235919906](juc.assets/image-20210711235919906.png) 

#### 竟态条件 Race Condition

多个线程在临界区内执行，由于代码的执行序列不同而导致结果无法预测，称之为发生了 **竟态条件** 

### synchronized 解决方案

#### <font color=" #28FF28">*案例-应用互斥</font> 

![image-20210712072242835](juc.assets/image-20210712072242835.png) 

#### synchronized语法

![image-20210712082727317](juc.assets/image-20210712082727317.png) 

![image-20210712072658102](juc.assets/image-20210712072658102.png) 

**图解**

![image-20210712075504295](juc.assets/image-20210712075504295.png)![image-20210712075616367](juc.assets/image-20210712075616367.png) 

synchronized 实际说用 **对象锁** 保证了 **临界区内代码的原子性** ，临界区内的代码对外是不可分割的，不会被线程切换所打断

### 方法上的synchronized

#### 成员方法

![image-20210712083136718](juc.assets/image-20210712083136718.png) 

#### 静态方法

![image-20210712083204943](juc.assets/image-20210712083204943.png) 

#### 线程八锁

考察synchronized锁住的是哪个对象

- **情况一 : 锁住this** 

  **现象** ： 直接打印1 2 或 直接打印 2 1

  ![image-20210712083449169](juc.assets/image-20210712083449169.png) 

- **情况二 : 锁住this** 

  **现象** ： 一秒后打印1再打印2 或 先打印2一秒后打印1

  ![image-20210712083811199](juc.assets/image-20210712083811199.png) 

- **情况三 ： 锁住a,b 的this对象**

  **现象**：3 一秒后1 2 或 3 2 一秒后 1 或 2 3 一秒后 1

  ![image-20210712084328060](juc.assets/image-20210712084328060.png) 

- **情况四 ：锁住不同对象的this**

  **现象** ： 先打印2一秒后打印1

  ![image-20210712085053314](juc.assets/image-20210712085053314.png) 

- **情况五 ：a锁住class对象， b锁住this**

  **现象** ： 先打印2一秒后打印1

  ![image-20210712085534779](juc.assets/image-20210712085534779.png) 

- **情况六 ：锁住class对象**

  **现象** ：一秒后打印1再打印2 或 先打印2一秒后打印1

  ![image-20210712085912808](juc.assets/image-20210712085912808.png) 

- **情况七 ：a锁住n1的class , b 锁住n2的this**

  **现象**：先打印2一秒后打印1

  ![image-20210712090039464](juc.assets/image-20210712090039464.png) 

- **情况八 ：锁住class对象**

  **现象** ：一秒后打印1再打印2 或 先打印2一秒后打印1

  ![image-20210712090427555](juc.assets/image-20210712090427555.png) 

### 变量的线程安全分析

#### 成员变量与静态变量是否线程安全？

- 如果他们没有共享，则线程安全
- 如果他们被共享了，根据他们的状态是否能够改变又分为两种情况
  - 如果只有读操作，则线程安全
  - 如果有读写操作，则这段代码是临界区，需要考虑线程安全

#### 局部变量是否线程安全？

- 局部变量是线程安全的
- 局部变量的引用对象则未必
  - 如果该对象没有逃离方法的作用访问，则线程安全
  - 如果该对象逃离方法的作用范围，需要考虑线程安全

#### 局部变量线程安全分析

- **局部变量** 

  ![image-20210712092130289](juc.assets/image-20210712092130289.png) 

  ![image-20210712092346562](juc.assets/image-20210712092346562.png) 

- 成员变量

  ![image-20210712092533398](juc.assets/image-20210712092533398.png) 

  ![image-20210712092728890](juc.assets/image-20210712092728890.png) 

  ![image-20210712092934858](juc.assets/image-20210712092934858.png) 

- 局部变量引用

  当访问修饰符为 *private* 

  ![image-20210712093158964](juc.assets/image-20210712093158964.png) 

  ![image-20210712093256155](juc.assets/image-20210712093256155.png) 

  当访问修饰符为 *public*

  ![image-20210712093911668](juc.assets/image-20210712093911668.png) 

  **情况一不会产生线程安全问题，但情况二不同，会导致list变为共享资源，从而产生线程安全问题**

#### 常见线程安全类

![image-20210712094754796](juc.assets/image-20210712094754796.png) 

##### 线程安全类的组合-- 可能线程不安全

![image-20210712095113805](juc.assets/image-20210712095113805.png) 

##### 不可变类线程的安全性 -- 线程安全

![image-20210712095414856](juc.assets/image-20210712095414856.png) 

