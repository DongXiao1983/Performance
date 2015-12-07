
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
