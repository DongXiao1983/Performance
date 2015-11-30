##DISKs

###1 Terminology
**Virtual disk**  
**Transport**  
**Sector**  
**I/O**  
**Disk Commands**  
**Throughput:** throughput commonly refers to the current data transfer rate, measured in bytes/second.    
**Bandwitch**  
**I/O latency:** 
**Lantency outliers:** disk I/O with unusually high latency    

###2 Models 
####2.1 Disk Types
	Simple disk   
	Cache disk: 磁盘上的write-back cache有利于提高磁盘的写性能   
	Controller disk : Host bus adaptors (HBAs)
####2.2 Concepts
	Measuring Time : Service time, Wait time.   
	IOPS :   
	Time Scales:   
	Caching :  Device cache ( ZFS vdev ) , Block cache ( buffer cache ), Disk controller cache ( RAID card cache ), Storage array cahe ( array cache ), On-disk cache ( disk data controller)   
####2.3 I/O latencies  
Noop: doesn't perform scheduling ( best choise for memory-backed block device : Ramdisk)   
Deadline:   
Anticipatory:An enhanced version of deadline  
CFQ: can use **ionice**  


###3 Methodology
- Tools method
- USE method
- Performance monitoring
- Workload characterization
- Latency analysis
- Event tracing
- Static performance tuning
- Cache tuning
- Resource controls
- Micro-benchmarking
- Scaling

建议的顺序： USE method -> Performance monitoring -> Workload characterization  -> Latency analysis -> Micro-benchmarking -> static analysis -> Event tracing  

####3.1 Tools Method
- **iostat**
- **iotop**
- **dtrace/stap/perf**: *iosnoop*
- **disk controller-specific tools**
####3.2 USE Method
1. the time the device was busy
2. the degree to which I/O is waiting in a queue
3. device errors
