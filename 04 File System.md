
# File System 

###1  Terminology  
- **File system**: an organization of data as files and directories, with a file-based interface for accessing them, and file permissions to control access. Additional content may include special file types for devices, sockets, and pipes, and metadata including file access timestamps.
- **File system cache**: an area of main memory (usually DRAM) used to cache file system contents, which may include different caches for various data and metadata types.
- **Operations**: File system operations are the requests of the file system, including read(), write(), open(), close(), stat(),mkdir(), and other operations.
- **I/O**: input/output. File system I/O can be defined in several ways; here it is used to mean only operations that directly read and write
- **(performing I/O)**, including read(), write(), stat() (read statistics), and mkdir() (write a new directory entry). I/O does not include open() and close().
- **Logical I/O**: I/O issued by the application to the file system.
- **Physical I/O**: I/O issued directly to disks by the file system (or via raw I/O).
- **Throughput**: the current data transfer rate between applications and the file system, measured in bytes per second.
- **inode**: An index node (inode) is a data structure containing metadata for a file system object, including permissions, timestamps, and data pointers.
- **VFS**: virtual file system, a kernel interface to abstract and support different file system types. On Solaris, a VFS inode is called a vnode.
- **Volume manager**: software for managing physical storages devices in a flexible way, creating virtual volumes from them for use by the OS.  


###2 Concepts

**File System Latency**   

**Caching**  

   
- **Page cache**:  operating system page cache  
- **File system primary cache**: ZFS ARC  
- **File system secondary cache**: ZFS L2ARC  
- **Directory cache**: directory cache, DNLC  
- **inode cache**:   inode cache  
- **Device cache**:   ZFS vdev   
- **Block device ache**:  buffer cache  

Random versus Sequential I/O  
Prefetch   
Read-Ahead   
Write-Back Caching   
Synchronous Writes : Individual Synchronous Writes ,  SynchronouslyCommitting Previous Writes    
Raw and Direct I/O : Raw I/O ,  Direct I/O    
Non-Blocking I/O  
Memory-Mapped Files   
Metadata   
Logical versus Physical I/O    
Special File Systems  


###3 Architecture
####3.1 Linux I/O stack   
![](https://www.thomas-krenn.com/de/wikiDE/images/b/ba/Linux-storage-stack-diagram_v4.0.png)

**File System I/O stack**  
**VFS(vNode)**  
####3.2 File System Cache
![](https://software.intel.com/sites/default/files/managed/ba/58/vfs1.png)   
**Buffer Cache**:   
**Page Cache**:  

- After an interval  
- The sync(), fsync() , or msync() system calls  
- Too many dirty pages(dirty_ratio)  
- No available pages in the page cache  

   kswapd,find and schedule dirty pages to be writeen to disk. the kswapd and flush threads are visible as kernel tasks for OS performance tools.  

**Dentry Cache**   

  &nbsp;&nbsp;The dentry cache (Dcache) remembers mappings from directory entry (struct dentry) to VFS inode, much like the earlier Unix DNLC. This improves the performance of path name lookups (e.g., via open()), as when a path name is traversed, each name lookup can check the Dcache for a direct inode mapping, instead of stepping through the directory contents. The Dcache entries are stored in a hash table for fast and scalable lookup (hashed by the parent dentry and directory entry name).

  &nbsp;&nbsp;Performance has been further improved over the years, including with the read-copy-update-walk (RCU-walk) algorithm [1]. This attempts to walk the path name without updating dentry reference counts, which were causing scalability issues due to cache coherency with high rates on multi-CPU systems. If a dentry is encountered that isn’t in the cache, RCU-walk reverts to the slower reference-count walk (ref-walk), since  reference counts will be necessary during file system lookup and blocking. For busy workloads, it’s expected that the dentrys will likely be cached, and so the RCU-walk approach will succeed.  
  &nbsp;&nbsp;The Dcache also performs negative caching, which remembers lookups for nonexistent entries. This improves the performance of failed lookups, which commonly occur for library path lookup.  
  &nbsp;&nbsp;The Dcache grows dynamically, shrinking via LRU when the system needs more memory. Its size can be seen via /proc.   
**Inode Cache**  

####3.2 File System Features 
**Block versus Extent**  
**Journaling**  
**Copy-on-Write** 

1. Write blocks to a new location (a new copy).  
2. Update references to new blocks.  
3. Add old blocks to the free list.  

**Scrubbing**  

####3.3 File System Types
FFS  
UFS  
ext3  
ext4  
ZFS  
btrfs  
####3.4 Volumes and Pools
&nbsp;&nbsp;Volumes present multiple disks as one virtual disk, upon which the file system is built. When built upon whole disks (and not slices or partitions), volumes isolate workloads, reducing performance issues of contention.   
&nbsp;&nbsp;Volume management software includes the Logical Volume Manager (LVM) for Linux-based systems, and the Solaris Volume Manager (SVM). Volumes, or virtual disks, may also be provided by hardware RAID controllers.   
&nbsp;&nbsp;Pooled storage includes multiple disks in a storage pool, from which multiple file systems can be created. Pooled storage is more flexible than volume storage, as file systems can grow and shrink regardless of the backing devices. This approach is used by modern file systems, including ZFS and btrfs.    
**Stripe width:** matching this to the workload.   
**Observability:** The virtual device utilization can be confusing; check the separate physical devices.   
**CPU overhead:** especially when performing RAID parity computation. This has become less of an issue with modern, faster CPUs.   
**Rebuilding:** Also called resilvering, this is when an empty disk is added to a RAID group (e.g., replacing a failed disk), and it is populated with the necessary data to join the group. This can significantly affect performance as it consumes I/O resources and may last for hours or even days.      
