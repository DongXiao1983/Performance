CPUs
---
1.	**Background**  
2.	**Architecture**  
3.	**Methodology**  
4.	**Analysis**  
5.	**Tunning**
    
       
1	 Backgroud    
	Terminology: Processor , core, Hardware, CPU instrauction, Logical CPU, Scheduler, Run queue  
	
1.1  Models :   
>	CPU Arch ( MMUN, MUNA )  
>	CPU Memory Caches  
>	CPU Run Queues  

1.2	Concepts  
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


2 Achitecture  
	Hardware: Processor ( P-cache, W-cache, Clock, Timestamp counter , Microcode Rom, Temperature senseors, Network interfaces )  
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

&emsp;
