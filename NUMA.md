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

#### Types of NUMA faults

- Locality   
	- “Local fault” memory on same node as CPU
	- “Remote fault” memory on different node than CPU
- Private vs shared   
	- “Private fault” memory accessed by same task twice in a row
	- “Shared fault” memory accessed by different task than last time


#### Task placement


- Best place to run a task   
   Where most of its memory accesses happen
- Best place to run a task   
   Where most of its memory accesses happen   
- It is not that simple   
	- Tasks may share memory  
  Some private accesses, some shared accesses  
  60% private, 40% shared is possible – group tasks together for best performance  
	- Tasks with memory on the node may have more threads than can run in one node's CPU cores
	- **Load balance may have spread threads across more physical CPUs**   <---- important !!!!!    
  Take advantage of more CPU cache   

#### Task placement constraints
- NUMA task placement may not create a load imbalance    
	- The load balancer would move something else   
	- Conflict can lead to tasks “bouncing around the system”  
      Bad locality  
      Lots of NUMA page migrations  
- NUMA task placement may
	- Swap tasks between nodes
	- Move a task to an idle CPU if no imbalance is created

####  Task placement algorithm  
For task A, check each NUMA node N
1. Check whether node N is better than task A's current node (C)  
  Task A has a larger fraction of memory accesses on node N, than on current node C  
  Score is the difference of fractions  
2. If so, check all CPUs on node N  
   Is the current task (T) on CPU better off on node C?  
   Is the CPU idle, and can we move task A to the CPU?  
   Is the benefit of moving task A to node N larger than the downside of moving task T to node C?  
3.For the CPU with the best score, move task A (and task T, to node C).


#### Task grouping
- Multiple tasks can access the same memory  
	- Threads in a large multi-threaded process (JVM, virtual machine, ...)
	- Processes using shared memory segment (eg. Database)  
- Use CPU num & pid in struct page to detect shared memory
	- At NUMA fault time, check CPU where page was last faulted
	- Group tasks together in numa_group, if PID matches
- Grouping related tasks improves NUMA task placement
	- Only group truly related tasks
	- Only group on write faults, ignore shared libraries like libc.so  
- Group stats are the sum of the NUMA fault stats for tasks in group
- Task placement code similar to before
- If a task belongs to a numa_group, use the numa_group stats for  comparison instead of the task stats
	- Pulls groups together, for more efficient access to shared memory
- When both compared tasks belong to the same numa_group
	- Use task stats, since group numbers are the same
	- Efficient placement of tasks within a group 


###3 NUMA performance
###4 NUMA tuning
