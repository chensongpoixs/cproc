<font face="微软雅黑"  >

### 一， linux内存管理机制

<font color = red>

linux虚拟内存实现了6种机制：


1. 地址映射机制
2. 内存分配回收机制
3. 缓存和刷新机制
4. 请求页机制
5. 交换机制
6. 内存共享机制

</font>

>内存管理理程序通过映射机制把⽤用户需要的逻辑地址映射到物理理地址，当⽤用户程序运⾏行行
时，如果发现程序中要⽤用的虚拟地址没有对应的物理理内存，就发出⻚页⾯面请求。如果有
空闲的内存分配，就请求分配（于是⽤用到了了内存分配和回收），并把正在使⽤用的物理理
内存记录在缓存中（使⽤用了了缓存机制）。如果没有⾜足够的内存提供分配，那么就调⽤用
交换机制，腾出内存。把物理理页写⼊入⽂文件中，并修改⻚页表来映射⽂文件。

- 1.每个进程的4G内存空间只是虚拟内存空间，每次访问内存空间的某个地址，都需要
把地址翻译为实际物理理地址

- 2.所有进程共享同⼀一物理理内存，每个进程只把⾃自⼰己⽬目前需要的虚拟内存空间映射并存
储到物理理内存上

- 3.进程要知道哪些内存地址上的数据在物理理内存上，哪些不不在，还有在物理理内存上的
哪⾥里里，需要⻚页表记录

- 4.页表的每⼀一个表项分为两部分，第⼀一部分记录此⻚页是否在物理理内存上，第⼆二部分记
录物理理内存的地址

- 5.当进程访问某个虚拟地址，去查看⻚页表，如果对应的数据不不在物理理内存中，，则缺
⻚页异常

- 6.缺⻚页异常的处理理过程，就是把进程需要的数据从磁盘拷⻉贝到物理理内存中，如果内存
已经满了了 ，没有空地⽅方，那就找⼀一个⻚页进⾏行行覆盖，当然如果被覆盖的⻚页曾经被修改
过，需要将此⻚页写回磁盘







### 2 、linux任务调度机制；

内核的三种调度策略略：

- a）SCHED_OTHER分时调度策略略；（默认的）

- b）SCHED_FIFO实时调度策略略，先到先服务；

- c）SCHED_RR实时调度策略略，时间⽚片轮转；





 实时进程将得到优先调用，实时进程根据实时优先级决定调度权值，分时进程则通过nice和counter值决定权值，nice越小，counter越大，被调度的概率越大，也就是曾经使用了cpu最少的进程将会得到优先调度。

<font color = red>
#### SHCED_RR和SCHED_FIFO的不同：
</font>

当采用SHCED_RR策略的进程的时间片用完，系统将重新分配时间片，并置于就绪队列尾。放在队列尾保证了所有具有相同优先级的RR任务的调度公平。  

SCHED_FIFO一旦占用cpu则一直运行。一直运行直到有更高优先级任务到达或自己放弃。
如果有相同优先级的实时进程（根据优先级计算的调度权值是一样的）已经准备好，FIFO时必须等待该进程主动放弃后才可以运行这个优先级相同的任务。而RR可以让每个任务都执行一段时间。


<font color = red>

##### 相同点：
</font>
- RR和FIFO都只用于实时任务。

- 创建时优先级大于0(1-99)。
- 按照可抢占优先级调度算法进行。
- 就绪态的实时任务立即抢占非实时任务。

#### 当所有任务都采用FIFO调度策略时（SCHED_FIFO）：

- 1.创建进程时指定采用FIFO，并设置实时优先级rt_priority(1-99)。

- 2.如果没有等待资源，则将该任务加入到就绪队列中。

- 3.调度程序遍历就绪队列，根据实时优先级计算调度权值,选择权值最高的任务使用cpu，该FIFO任务将一直占有cpu直到有优先级更高的任务就绪(即使优先级相同也不行)或者主动放弃(等待资源)。

- 4.调度程序发现有优先级更高的任务到达(高优先级任务可能被中断或定时器任务唤醒，再或被当前运行的任务唤醒，等等)，则调度程序立即在当前任务堆栈中保存当前cpu寄存器的所有数据，重新从高优先级任务的堆栈中加载寄存器数据到cpu，此时高优先级的任务开始运行。重复第3步。

- 5.如果当前任务因等待资源而主动放弃cpu使用权，则该任务将从就绪队列中删除，加入等待队列，此时重复第3步。




#### 当所有任务都采用RR调度策略（SCHED_RR）时：

1.创建任务时指定调度参数为RR，并设置任务的实时优先级和nice值(nice值将会转换为该任务的时间片的长度)。

2.如果没有等待资源，则将该任务加入到就绪队列中。

3.调度程序遍历就绪队列，根据实时优先级计算调度权值,选择权值最高的任务使用cpu。

4.如果就绪队列中的RR任务时间片为0，则会根据nice值设置该任务的时间片，同时将该任务放入就绪队列的末尾。重复步骤3。

5.当前任务由于等待资源而主动退出cpu，则其加入等待队列中。重复步骤3。


#### 系统中既有分时调度，又有时间片轮转调度和先进先出调度：

- 1.RR调度和FIFO调度的进程属于实时进程，以分时调度的进程是非实时进程。

- 2.当实时进程准备就绪后，如果当前cpu正在运行非实时进程，则实时进程立即抢占非实时进程。

- 3.RR进程和FIFO进程都采用实时优先级做为调度的权值标准，RR是FIFO的一个延伸。FIFO时，如果两个进程的优先级一样，则这两个优先级一样的进程具体执行哪一个是由其在队列中的未知决定的，这样导致一些不公正性(优先级是一样的，为什么要让你一直运行?),如果将两个优先级一样的任务的调度策略都设为RR,则保证了这两个任务可以循环执行，保证了公平。








进程的task_struct结构有以下四个选项：policy、priority（nice值）、
counter、rt_priority。其中policy是进程的调度策略略，⽤用来区分实时进程和普
通进程，实时进程优先于普通进程运⾏行行，priority是线程的优先级，counter是进
程的剩余时间⽚片。rt_priority是实时进程持有的，⽤用于实时进程的选择。

其中：nice -n可以设置程序的优先级
</font>