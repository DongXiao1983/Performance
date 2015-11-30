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
For disk Devices :  
&nbsp;  1. the time the device was busy  
&nbsp;  2. the degree to which I/O is waiting in a queue  
&nbsp;  3. device errors  

For disk controllers:  
&nbsp;  1. current versus maximum throughput, and the same for operation rate  
&nbsp;  2. the degree to which I/O is waitting due to controller saturation  
&nbsp;  3. controller errors  

**iostat** do not present per-controller metrcs but provide only per disk.

####3.3 Performance monitoring
Disk utilization   
Response time  

Disk utilization over 60% may cause poor performance due to increased queueing.  

####3.4 Workload characterization 
Disk I/O workload characterizing:

- I/O rate  
- I/O throughput  
- I/O size  
- Ranom versus sequential  
- Read/write ratio  

Workload characterization check list :

- What is the IOPS rate system-wide? Per disk? Per controller?  
- What is the throughput system-wide? Per disk? Per controller?  
- Which applications or users are using the disks?  
- What file systems or files are being accessed?  
- Have any errors been encountered? Were they due to invalid requests, or issues on the disk?  
- How balanced is the I/O over available disks?  
- What is the IOPS for each transport bus involved?  
- What is the throughput for each transport bus involved?  
- What non-data-transfer disk commands are being issued?  
- Why is disk I/O issued (kernel call path)?  
- To what degree is disk I/O application-synchronous?  
- What is the distribution of I/O arrival times?    

Performance Characterization: 

- How busy is each disk (utilization)?  
- How saturated is each disk with I/O (wait queueing)?  
- What is the average I/O service time?  
- What is the average I/O wait time?  
- Are there I/O outliers with high latency?  
- What is the full distribution of I/O latency?  
- Are system resource controls, such as I/O throttling, present and active?  
- What is the latency of non-data-transfer disk commands?  

####3.5 Latency analysis
The latency at each level may be presented as  

- Per-interval I/O averages: as typically reported by operating system tools  
- Full I/O distributions: as histograms or heat maps
- Per-I/O latency values: see the next section, Event Tracing    

####3.6 Event Tracing  
  
- **Disk device ID**   
- **I/O type**: read or write   
- **I/O offset**: disk location   
- **I/O size**: bytes   
- **I/O request timestamp**: when an I/O was issued to the device (also known as an I/O strategy)   
- **I/O completion timestamp**: when the I/O event completed (completion interrupt)   
- **I/O completion status**: errors   

Additional details may include, when applicable, PID, UID, application name, file name, and events for all non-data-transfer disk commands (and
custom details for those commands).  

####3.7 Static Performance Tuning   
- How many disks are present? Of which types?  
- What version is the disk firmware?  
- How many disk controllers are present? Of which interface types?  
- Are disk controller cards connected to high-speed slots?  
- What version is the disk controller firmware?  
- Is RAID configured? How exactly, including stripe width?  
- Is multipathing available and configured?  
- What version is the disk device driver?  
- Are there operating system bugs/patches for any of the storage device drivers?  
- Are there resource controls in use for disk I/O?  

####3.8 Static Performance Tuning  
- How many disks are present? Of which types?
- What version is the disk firmware?
- How many disk controllers are present? Of which interface types?
- Are disk controller cards connected to high-speed slots?
- What version is the disk controller firmware?
- Is RAID configured? How exactly, including stripe width?
- Is multipathing available and configured?
- What version is the disk device driver?
- Are there operating system bugs/patches for any of the storage device drivers?
- Are there resource controls in use for disk I/O?   

###4 Analysis
**iostat**  various per-disk statistics  
**sar** historical disk statistics  
**pidstat, iotop**  disk I/O usage by process  
**blktrace**  disk I/O event tracing   
**DTrace**    custome static and dynamic tracing     
**smartctl**  disk controller statistic  

####4.1 iostat
summarizes per-disk I/O statistics.
