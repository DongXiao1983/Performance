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
>	Clock Rate   
>	Instraction : fetch, decode, execute, memory access, register write-back  
>	Instraction Pipline   
>	Instruction Width  
>	CPI,IPC  
>	Utilization  
>	User-Time/Kernel-Time  
>	Sarturation 
>	Preemption  
>	Priority Inversion  
>	Multiprocess, Multithreading  
>	Word Size  
>	Compiler Optimization  


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

&emsp;
