CPUs
---
1.	**Background**  
2.	**Architecture**  
3.	**Methodology**  
4.	**Analysis**  
5.	**Tunning**
    
       
###1	 Backgroud    
	Terminology: Processor , core, Hardware, CPU instrauction, Logical CPU, Scheduler, Run queue  
	
####1.1  Models :   
>	CPU Arch ( MMUN, MUNA )  
>	CPU Memory Caches  
>	CPU Run Queues  

####1.2	Concepts  
	Clock Rate   
	Instraction : fetch, decode, execute, memory access, register write-back  
	Instraction Pipline   
	Instruction Width  
	CPI,IPC  
	Utilization  
	User-Time/Kernel-Time  
	Sarturation 
	Preemption  
	Priority Inversion  
	Multiprocess, Multithreading  
	Word Size  
	Compiler Optimization  


###2 Achitecture  
##### 2.1 Hardware:
 Processor：  ( P-cache, W-cache, Clock, Timestamp counter , Microcode Rom, Temperature senseors, Network interfaces )  
	CPU Cache:  

	Level instrauction chace(I$)     
	Level data cache(D$)    
	Translation lookaside buffer(TLB)   
	Level 2 cache (E$)   
	Level 3 cache  


Associativity:  Fully, Direct mapped , Set  
Cache line  
Cache Coherency   
MMU    
Interconnects : UMA  NUMA   
	   CPU Performance Counters:  

	CPU cycles: including stall cycles and types of stall cycles   
	CPU instrauctions: retired (executed )   
	Level 1,2,3 chace accesses: hits, miss   
	Floating-point unit: operations    
	Memory I/O: reads , writes, stall cycles   
	Resource I/O: reads, writes, stall cycles   

 
 CPU计数器  Unit Mask  

##### 2.2 Software
Sechedule :  
  
	Time Sharing
	Preemption   
	Load balancing  

Functions:  
      
    scheduler_tick()
    check_preemot_curr()
    __Scheduler()
    pick_next_task()
    load_balance()
 
Secheduling Classes

    RT: user & kernel level preemption  0-99(MAX_RT_PRIO-1)
	O(1): scheduler dynamincaly improves the proority of I/O-bound over CPU_bound workloads. to reduce latency of interative and I/O workloads.
	CFS:   

&emsp; schedsetcheduler() 设置用户态的调度行为，RT支持 SCHED_RR & SCHED_FIFO, CFS支持SCHED_NORMAL & SCHED_BATCH  
    
  关于调度算法，可以参考：  
&emsp;" Hyper-Threading Aeware Process Scheduling Heuristics" USENIX,2005   
&emsp;"Solaris Performance and Tools: DTrace and MDB Techiques for Solaris 10 and open Solaris."    

Idle Thread   
NUMA Grouping:  making the kernel NUMA-aware, make better scheduling and
memory placement decisions   
Tools:  [http://minnie.tuhs.org/cgi-bin/utree.pl?file=V4](http://minnie.tuhs.org/cgi-bin/utree.pl?file=V4)   

###3 Methodology   
- Tools method
- USE method 
- Workload characterizaion 
- Profiling
- Cycle analysis
- Performance monitoring
- Static performance tuning
- Priority tuning
- Resource controls
- CPU binding
- Micro-benchmarking
- Scaling  

建议的顺序： Performance monitoring -> USE method -> Profiling -> Micro-benchmarking -> s
tatic analysis

####3.1 Tools Method
- **uptime**:
- **vmstat**: `watch -n 1 -d vmstat`  如果id低于10 ，一般有问题。
- **mpstat**: `mpstat -P ALL` 查看CPU上的调度有没有问题，是否存在特别忙的CPU
- **top/prstat**:
- **pidstat/prstat**: 产看进程的用户时间，内核时间，跑在哪个CPU上
- **perf/dtrace/stap/oprofile**:
- **perf/cpustat**:

####3.2 Tools Method  
每个CPU主要检查这三项：
   
	Utilization：CPU在非空闲线程上的忙碌时间
	Saturation： CPU上跑的进程队列的等待时间
	Errors：  

####3.3 Workload characterizaion
Basic checklist：   

- Load averages ( utilization + saturation )
- User-time to system-time ratio
- Syscall rate
- Voluntary context switch rate
- Interrupt rate

Advanced checklist:  

- What is the CPU utilization system-wide? Per CPU?
- How parallel is the CPU load? Is it single-threaded? How many threads?
- Which applications or users are using the CPUs? How much?
- Which kernel threads are using the CPUs? How much?
- What is the CPU usage of interrupts?
- What is the CPU interconnect utilization?
- Why are the CPUs being used (userand kernel-level call paths)?
- What types of stall cycles are encountered?

####3.4 Cycle analysis

	memory-I/O-intensive   
	instruction-intensive 

参考软件：“The Oracle Solaris Studio Performance Tools”

####3.5 Performance monitoring

	Utilization : percent busy
	Saturation: run-queue length, inferred from load average, thread scheduler latnecy

难度在于测量的时间间隔，对于burst的触发，间隔太长观测不到，太短不准确。一般倾向1s。 
 
####3.6 Static performance tuning
静态分析手段一般关注在CPU的配置面，一般有以下几个方面：  

- How many CPUs are available for use? Are they cores? Hardware threads?
- Is the CPU architecture singleor multiprocessor?
- What is the size of the CPU caches? Are they shared?
- What is the CPU clock speed? Is it dynamic (e.g., Intel Turbo Boost and SpeedStep)? Are those dynamic features enabled in the BIOS?
- What other CPU-related features are enabled or disabled in the BIOS?
- Are there performance issues (bugs) with this processor model? Are they listed in the processor errata sheet?
- Are there performance issues (bugs) with this BIOS firmware version?
- Are there software-imposed CPU usage limits (resource controls) present? What are they?
####3.7 Priority tuning
 命令 `nice`， 负数优先级高。   
重点关注那些实时调度 real-time scheduling class。
####3.8 CPU Binding
在NUMA中能够改善本地memory的存取性能。 两种手段：  

- Process binding:
- Exclusive CPU sets: 主要用于提高CPU的cache性能（cache warm）

命令 `cpuset`

###4  Analysis

<tr>   <td>`uptime` </td>   <td>load averages</td></tr> 
<tr>   <td>`vmstat` </td>   <td>includes system-wide CPU averages</td></tr> 
<tr>   <td>`mpsta`t </td>   <td>per-CPU statistics</td></tr> 
<tr>   <td>`sar` </td>   <td>historical statistics</td></tr> 
<tr>   <td>`ps`  </td>   <td>process status</td></tr> 
<tr>   <td>`top`  </td>   <td>monitor per-process/thread CPU usage</td></tr> 
<tr>   <td>`pidstat`   </td>   <td>per-process/thread CPU breakdowns</td></tr> 
<tr>   <td>`time`  </td>   <td>process status</td></tr> 
<tr>   <td>`DTrace/perf`  &emsp;</td>   <td>CPU profiling & trace</td></tr> 
###5  Tuning
目标1, 识别出CPU不需要的的消耗  
目标2, binding CPU
 
####5.1  Compiler option
####5.2  Scheduling Priority and Class
命令`nice`，调整进程的优先级。(-20,+19)，-20d 的优先级最高。  
命令`chrt`，设置进程的调度策略。  

    compute-1:~$ chrt -m
    SCHED_OTHER min/max priority: 0/0
    SCHED_FIFO min/max priority : 1/99
    SCHED_RR min/max priority   : 1/99
    SCHED_BATCH min/max priority: 0/0
    SCHED_IDLE min/max priority : 0/0

代码 `setpriority(), sched_setcheduler()`
####5.3 Scheduler Options
kernel 编译参数，在`/boot/config-xxxxx`里  

    CONFIG_CGROUP_SCHED
    CONFIG_FAIR_GROUP_SCHED
    CONFIG_RT_GROUP_SCHED
    CONFIG_SCHED_AUTOGROUP
    CONFIG_SCHED_SMT
    CONFIG_SCHED_MC
    CONFIG_HZ
    CONFIG_NO_HZ
    CONFIG_SCHED_HRTICK
    CONFIG_PREEMPT
    CONFIG_PREEMPT_NONE
    CONFIG_PREEMPPT_VOLUNTARY   


系统的配置文件：  

    compute-1:~$ ls -la /proc/sys/kernel/sched_
    sched_cfs_bandwidth_slice_us  sched_rt_period_us
    sched_child_runs_first        sched_rt_runtime_us
    sched_domain/                 sched_rt_throttle_warn
    sched_latency_ns              sched_shares_window_ns
    sched_migration_cost_ns       sched_time_avg_ms
    sched_min_granularity_ns      sched_tunable_scaling
    sched_nr_migrate              sched_wakeup_granularity_ns
    sched_rr_timeslice_ms

####5.4 Process Binding   

命令 `taskset`   
linux 也提供了cpuset的挂载修改  

    # mkdir /dev/cpuset
    # mount -t cpuset cpuset /dev/cpuset
    # cd /dev/cpuset
    # mkdir prodset # create a cpuset called "prod
    # cd prodset
    # echo 7-10 > cpus # assign CPUs 7-10
    # echo 1 > cpu_exclusive # make prodset exclusive
    # echo 1159 > tasks # assign PID 1159 to prodset
