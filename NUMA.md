
##NUMA Balancing

###1 What's NUMA
- Non Uniform Memory Access     
- Multiple physical CPUs in a system     
- Each CPU has memory attached to it  
- Each CPU can access other CPU's memory, too  

#### NUMA terminology

- Node  
	- A physical CPU and attached memory  
	- Could be multiple CPUs (with off-chip memory controller)  
- Interconnect   
	- Bus connecting the various nodes together  
	- Generally faster than memory bandwidth of a single node  
	- Can get overwhelmed by traffic from many nodes  

####  NUMA performance considerations
- NUMA performance penalties from two main sources
	- Higher latency of accessing remote memory
	- Interconnect contention
- Processor threads and cores share resources
	- Execution units (between HT threads)
	- Cache (between threads and cores)   

#### NUMA balancing strategies
- CPU follows memory
	- Try running tasks where their memory is
- Memory follows CPU
	- Move memory to where it is accessed
- other strategies 
	- Various mechanisms involved
	- Lots of interesting corner cases...

###2 NUMA internal

- NUMA hinting page faults
- NUMA page migration
- Task grouping
- Fault statistics
- Task placement
- Pseudo-interleaving

#### NUMA hinting page faults
- Periodically, each task's memory is unmapped
	- Period based on run time, and NUMA locality
	- Unmapped “a little bit” at a time (chunks of 256MB)
	- Page table set to “no access permission” marked as NUMA pte
- Page faults generated as task tries to access memory
	- Used to track the location of memory a task uses
		- Task may also have unused memory “just sitting around”
	- NUMA faults also drive NUMA page migration

#### NUMA page migration
- NUMA page faults are relatively cheap
- Page migration is much more expensive
-	 but so is having task memory on the “wrong node”
- Quadratic filter: only migrate if page is accessed twice
	- From same NUMA node, or
	- By the same task
	- CPU number & low bits of pid in page struct
- Page is migrated to where the task is running

####  Fault statistics
- Fault statistics are used to place tasks (cpu-follows-memory)   
- Statistics kept per task, and per numa_group    
- “Where is the memory this task (or group) is accessing?”   
	-  “NUMA page faults” counter per NUMA node    
	-  After a NUMA fault, account the page location   
	-  If the page was migrated, account the new location   
-  Kept as a floating average    

###3 NUMA performance
###4 NUMA tuning
