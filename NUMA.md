##NUMA Balancing

###1 What's NUMA
- Non Uniform Memory Access     
- Multiple physical CPUs in a system     
- Each CPU has memory attached to it  
- Each CPU can access other CPU's memory, too  

####1.1 NUMA terminology

- Node  
	- A physical CPU and attached memory  
	- Could be multiple CPUs (with off-chip memory controller)  
- Interconnect   
	- Bus connecting the various nodes together  
	- Generally faster than memory bandwidth of a single node  
	- Can get overwhelmed by traffic from many nodes  

####1.2  NUMA performance considerations
- NUMA performance penalties from two main sources
	- Higher latency of accessing remote memory
	- Interconnect contention
- Processor threads and cores share resources
	- Execution units (between HT threads)
	- Cache (between threads and cores)   

####1.2 NUMA balancing strategies
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

####2.1 NUMA hinting page faults
- Periodically, each task's memory is unmapped
	- Period based on run time, and NUMA locality
	- Unmapped “a little bit” at a time (chunks of 256MB)
	- Page table set to “no access permission” marked as NUMA pte
- Page faults generated as task tries to access memory
	- Used to track the location of memory a task uses
		- Task may also have unused memory “just sitting around”
	- NUMA faults also drive NUMA page migration

####2.2 NUMA page migration
- NUMA page faults are relatively cheap
- Page migration is much more expensive
-	 but so is having task memory on the “wrong node”
- Quadratic filter: only migrate if page is accessed twice
	- From same NUMA node, or
	- By the same task
	- CPU number & low bits of pid in page struct
- Page is migrated to where the task is running

####2.3  Fault statistics
- Fault statistics are used to place tasks (cpu-follows-memory)   
- Statistics kept per task, and per numa_group    
- “Where is the memory this task (or group) is accessing?”   
	-  “NUMA page faults” counter per NUMA node    
	-  After a NUMA fault, account the page location   
	-  If the page was migrated, account the new location   
-  Kept as a floating average    

####2.4 Types of NUMA faults

- Locality   
	- “Local fault” memory on same node as CPU
	- “Remote fault” memory on different node than CPU
- Private vs shared   
	- “Private fault” memory accessed by same task twice in a row
	- “Shared fault” memory accessed by different task than last time


####2.5 Task placement


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

####2.6 Task placement constraints
- NUMA task placement may not create a load imbalance    
	- The load balancer would move something else   
	- Conflict can lead to tasks “bouncing around the system”  
      Bad locality  
      Lots of NUMA page migrations  
- NUMA task placement may
	- Swap tasks between nodes
	- Move a task to an idle CPU if no imbalance is created

####2.7  Task placement algorithm  
For task A, check each NUMA node N
1. Check whether node N is better than task A's current node (C)  
  Task A has a larger fraction of memory accesses on node N, than on current node C  
  Score is the difference of fractions  
2. If so, check all CPUs on node N  
   Is the current task (T) on CPU better off on node C?  
   Is the CPU idle, and can we move task A to the CPU?  
   Is the benefit of moving task A to node N larger than the downside of moving task T to node C?  
3.For the CPU with the best score, move task A (and task T, to node C).


####2.8 Task grouping
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
####3.1 KVM guests
- Size of the guests continue to increase
• Use cases include classic enterprise scale-up guests & guests in private cloud environments
- [Manual] pinning/tuning of individual guests using libvirt/virsh (after taking into account host's NUMA topology) are required to achieve low overhead & predictable performance for workloads running in mid/large sized guests. This is especially true on larger scale-up hosts (e.g. >= 4 socket servers)  


	    <cputune>
	      <vcpupin vcpu='0' cpuset='0'/>
	      <vcpupin vcpu='1' cpuset='1'/>
	      <vcpupin vcpu='2' cpuset='2'/>
	      <vcpupin vcpu='3' cpuset='3'/>
	      <vcpupin vcpu='29' cpuset='29'/>
	    </cputune>

	    <numatune>
	      <memory mode='preferred' nodeset='0-1'/>
	    </numatune>


- its harder to live migrate such pinned guest across hosts – as similar set of backing resources may not be available on the target host (or) the target host may have a different NUMA topology !  

- Automatic NUMA balancing could help avoid the need for having to manually pin the guest resources
	- Exceptions include cases where a guest's backing memory pages on the host can't be migrated :
		- Guests relying on hugetlbfs (instead of THPs)
		- Guests with direct device assignment (get_user_pages())
		- Guests with real time needs (mlock_all()).
- As the guest size spans more than 1 socket it is highly recommended to enable Virtual NUMA nodes in the guest => helps the guest OS instance to scale/perform.


	    <cpu>
	      <topology sockets='2' cores='15' threads='1'/>
	      <numa>
	        <cell cpus='0-14' memory='134217728'/>
	        <cell cpus='15-29' memory='134217728'/>
	      </numa>
	    </cpu>

- Virtual NUMA nodes in the guest OS => Automatic NUMA balancing enabled in the guest OS instance.
	- Users can still choose to pin the workload to these Virtual NUMA nodes, if it helps their use case.

###4 NUMA unchecked feature
####4.1 Complex NUMA topologies & pseudo-interleaving
- Differing distances between NUMA nodes
	- Local node, nearby nodes, far away nodes
	- Eg. 20% & 100% performance penalty for nearby vs. far away
- Workloads that are spread across multiple nodes work better  
  when those nodes are near each other
- Unclear how to implement this  
  When is the second-best node no longer the second best?  
#### 4.2 NUMA balancing & unmovable memory

- Unmovable memory
	- Mlock
	- Hugetlbfs
	- Pinning for KVM device assignment & RDMA
- Memory is not movable ...
	- But the tasks are
	- NUMA faults would help move the task near the memory
	- Unclear if worthwhile, needs experimentation

####4.3 KSM  
- Kernel Samepage Merging
	- De-duplicates identical content between KVM guests
	- Also usable by other programs
- KSM has simple NUMA option
	- “Only merge between tasks on the same NUMA node”
	- Task can be moved after memory is merged
	- May need NUMA faults on KSM pages, and re-locate memory if needed
	- Unclear if worthwhile, needs experimentation

#### 4.4 Interrupt locality
- Some tasks do a lot of IO
	- Performance benefit to placing those tasks near the IO device
	- Manual binding demonstrates that benefit
- Currently automatic NUMA balancing only does memory & CPU
- Would need to be enhanced to consider IRQ/device locality
- Unclear how to implement this
	- When should IRQ affinity outweigh CPU/memory affinity?

#### 4.5 Inter Process Communication  
- Some tasks communicate a LOT with other tasks
	- Benefit from NUMA placement near tasks they communicate with
	- Do not necessarily share any memory
- Loopback TCP/UDP, named socket, pipe, ...
- Unclear how to implement this
	- How to weigh against CPU/memory locality?
