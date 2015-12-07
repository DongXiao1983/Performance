
#Memory 

###1  Terminology  
- **Main memory**: Also referred to as physical memory, this describes the fast data storage area of a computer, commonly provided as DRAM.   
- **Virtual memory**: an abstraction of main memory that is (almost) infinite and noncontended. Virtual memory is not real memory.   
- **Resident memory**: memory that currently resides in main memory.   
- **Anonymous memory**: memory with no file system location or path name. It includes the working data of a process address space, called the heap.   
- **Address space**: a memory context. There are virtual address spaces for each process, and for the kernel.   
- **Segment**: an area of memory flagged for a particular purpose, such as for storing executable or writeable pages.   
- **OOM**: out of memory, when the kernel detects low available memory.   
- **Page**: a unit of memory, as used by the OS and CPUs. Historically it is either 4 or 8 Kbytes. Modern processors have multiple page size support for larger sizes.     
- **Page fault**: an invalid memory access. These are normal occurrences when using on-demand virtual memory.   
- **Paging**: the transfer of pages between main memory and the storage devices.   
- **Swapping**: From Unix, this is the transfer of entire processes between main memory and the swap devices. Linux often uses swapping to refer to paging to the swap device (the transfer of swap pages). In this book the original definition is used: swapping is for entire processes.  
- **Swap**: an on-disk area for paged anonymous data and swapped processes. It may be an area on a storage device, also called a physical  
- **swap device**, or a file system file, called a swap file. Some tools use the term swap to refer to virtual memory (which is confusing and incorrect).  

###2 Architecture

Main MemoryArchitecture  

- Busses  
	- **Shared system bus**: single or multiprocessor, via a shared system bus, a memory bridge controller, and finally a memory bus. The memory controller in that example was a Northbridge.  
	- 	**Direct**: single processor with directly attached memory via a memory bus.   
	- 	**Interconnect**: multiprocessor, each with directly attached memory via a memory bus, and processors connected via a CPU interconnect.   

Memory allocator  

- **Slab** The kernel slab allocator manages caches of objects of a specific size, allowing them to be recycled quickly without the overhead of page allocation. This is especially effective for kernel allocations, which are frequently for fixed-size structs.  
- **SLUB** The Linux kernel SLUB allocator is based on the slab allocator and is designed to address various concerns, especially regarding the complexity of the slab allocator. These include the removal of object queues, and also per-CPU caches—leaving NUMA optimization to the page allocator
- **libc**   
- **glibc**


###3 Methodology
####3.1 Tools Method  
- **Page scanning**: Look for continual page scanning (more than 10 s) as a sign of memory pressure. On Linux, this can be done using "**sar -B**" and checking the pgscan columns.  
- **Paging**: The paging of memory is a further indication that the system is low on memory. **vmstat(8)** and check the **si** and **so** columns (here, the term swapping means anonymous paging).  
- **vmstat**: Run vmstat per second and check the free column for available memory.
- **OOM killer**: hese events can be seen in the system log **/var/log/messages**, or from **dmesg**(1). Search for “Out of memory.”  
- **top/prstat:** See which processes and users are the top physical memory consumers (resident) and virtual memory consumers (see the man page for the names of the columns, which differ depending on version). These tools also summarize free memory.  
- **dtrace/stap/perf**: Trace memory allocations with stack traces, to identify the cause of memory usage.  

####3.2 Characterizing Usage    
- System-wide physical and virtual memory utilization
- Degree of saturation: paging, swapping, OOM killing
- Kernel and file system cache memory usage
- Per-process physical and virtual memory usage
- Usage of memory resource controls, if present  

####3.3 Advanced Usage Analysis/Checklist

- Where is the kernel memory used? Per slab?
- How much of the file system cache (or page cache) is active as opposed to inactive?
- Where is the process memory used?
- Why are processes allocating memory (call paths)?
- Why is the kernel allocating memory (call paths)?
- What processes are actively being paged/swapped out?
- What processes have previously been paged/swapped out?
- May processes or the kernel have memory leaks?
- In a NUMA system, how well is memory distributed across memory nodes?
- What are the CPI and memory stall cycle rates?
- How balanced are the memory busses?
- How much local memory I/O is performed as opposed to remote memory I/O?  

####3.4 Static Performance Tuning 

- How much main memory is there in total?
- How much memory are applications configured to use (their own config)?
- Which memory allocators do the applications use?
- What is the speed of main memory? Is it the fastest type available?
- What is the system architecture? NUMA, UMA?
- Is the operating system NUMA-aware?
- How many memory busses are present?
- What are the number and size of the CPU caches? TLB?
- Are large pages configured and used?
- Is overcommit available and configured?
- What other system memory tunables are in use?
- Are there software-imposed memory limits (resource controls)?  

###4 Analysis
####4.1 vmstat

	compute-2:~$ sudo vmstat 1
	Password:
	procs -----------memory---------- ---swap-- -----io---- -system-- ----cpu----
	 r  b  swpd   free   buff   cache   si   so   bi    bo   in       cs us sy id wa
	17  0     0 2473940  18568 2224704  0     0   17    76   12       12  9  1 90  0
	13  0     0 2473288  18568 2224736  0     0    0   848 125315 331610 27  2 71  1
	14  0     0 2473280  18568 2224736  0     0    0   612 122073 327298 23  2 75  0
	14  0     0 2473312  18568 2224736  0     0    0   368 127829 342581 25  2 72  0
	18  0     0 2473496  18576 2224728  0     0    0   228 125591 338790 28  2 70  0
	13  0     0 2473036  18576 2224736  0     0    0  1148 125348 338968 26  2 71  0
	10  0     0 2471804  18584 2224736  0     0   72   568 126974 335950 26  2 72  0
	35  0     0 2471644  18584 2224736  0     0    0  3636 124512 328179 28  2 70  0

- **swpd**: amount of swapped-out memory
- **free**: free available memory
- **buff**: memory in the buffer cache
- **cache**: memory in the page cache
- **si**: memory swapped in (paging)
- **so**: memory swapped out (paging)  

If the **si** and **so** columns are continually nonzero, the system is under memory pressure and is paging to a swap device or file (see **swapon**(8))  

####4.2 sar 
- -B: paging statistics
- -H: huge pages statistics
- -r: memory utilization
- -R: memory statistics
- -S: swap space statistics
- -W: swapping statistics  

####4.3 slabtop  
In this example, the **-sc** option was used to sort by cache size, with the largest at the top.   
The slab statistics are from **/proc/slabinfo** and can also be printed using **`vmstat -m`**.   

	 Active / Total Objects (% used): 645312 / 688491 (93.7%)
	 Active / Total Slabs (% used)  : 17747 / 17747 (100.0%)
	 Active / Total Caches (% used) : 91 / 131 (69.5%)
	 Active / Total Size (% used)   : 172163.03K / 185844.12K (92.6%)
	 Minimum / Average / Maximum Object : 0.01K / 0.27K / 15.81K
	
	  OBJS ACTIVE  USE   OBJ SIZE  SLABS OBJ/SLAB CACHE SIZE NAME
	256311 234236  91%      0.24K   7767     33  62136K buffer_head
	 19964  11560  57%      0.55K    713     28  11408K radix_tree_node
	  2800   2770  98%      4.00K     35     08  11200K kmalloc-4096
	 10112   9829  97%      1.00K    316     32  10112K kmalloc-1024
	  1525   1403  92%      6.03K     30     55   9760K task_struct
	 35224  35054  99%      0.23K   1036     34   8288K dentry
	 68220  67877  99%      0.11K   1895     36   7580K sysfs_dir_cache
	 10011   9972  99%      0.67K    213     47   6816K inode_cache
	 26838  26037  97%      0.19K    639     42   5112K kmalloc-192
	  3650   3359  92%      1.24K    146     25   4672K ext4_inode_cache

####4.4 ps
####4.5 top
####4.6 prstat
####4.7 pmap
####4.8 others

- **free**: report free memory, with buffer cache and page cache (see Chapter 8, File Systems).
- **dmesg**: check for “Out of memory” messages from the OOM killer.
- **valgrind**: a performance analysis suite, including memcheck, a wrapper for user-level allocators for memory usage analysis including leak
- **detection**. This costs significant overhead; the manual advises that it can cause the target to run 20 to 30 times slower [3].
- **swapon**: to add and observe physical swap devices or files.
- **iostat**: If the swap device is a physical disk or slice, device I/O may be observable using iostat(1), which indicates that the system is paging.
- **perf**: Introduced in Chapter 6, CPUs, this can be used to investigate CPI, MMU/TSB events, and memory bus stall cycles from the CPU performance instrumentation counters. It also provides probes for page faults and several kernel memory (kmem) events.
- **/proc/zoneinfo**: statistics for memory zones (NUMA nodes).
- **/proc/buddyinfo**: statistics for the kernel buddy allocator for pages.

###5 Tuning
Various memory tunable parameters are described in the kernel source documentation in **Documentation/sysctl/vm.txt** and can be set using **sysctl(8)**  

Multiple Page Sizes:    
larger page,  see **Documentation/vm/hugetlbpage.txt**.  
  

	[root@heaven /usr/src/kernels]# echo 50 >  /proc/sys/vm/nr_hugepages
	[root@heaven /usr/src/kernels]# grep Huge /proc/meminfo
	AnonHugePages:575488 kB
	HugePages_Total:  50
	HugePages_Free:   50
	HugePages_Rsvd:    0
	HugePages_Surp:    0
	Hugepagesize:   2048 kB
    
    [root@heaven /usr/src/kernels]# mkdir /mnt/hugetlbfs
    [root@heaven /usr/src/kernels]# mount -t hugetlbfs none /mnt/hugetlbfs -o pagesize=2048K


Resource Controls  

Basic resource controls, including setting a main memory limit and a virtual memory limit, may be available using **ulimit(1)**.   

For Linux, the container groups (cgroups) memory subsystem provides various additional controls. These include     

- **memory.memsw.limit\_in_bytes**: the maximum allowed memory and swap space, in bytes   
- **memory.limit\_in_bytes**: the maximum allowed user memory, including file cache usage, in bytes   
- **memory.swappiness**: similar to vm.swappiness described earlier but can be set for a cgroup    
- **memory.oom_control**: can be set to 0, to allow the OOM killer for this cgroup, or 1, to disable it  
