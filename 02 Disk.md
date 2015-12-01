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

	compute-1:~$ iostat -xkdz 1
	Linux 3.10.84-ovp-rt89-r5_preempt-rt (compute-1)12/01/15   _x86_64_  (48 CPU)
	Device: rrqm/s  wrqm/s r/s    w/s  rkB/s  wkB/s avgrq-sz avgqu-sz await r_await w_await  svctm  %util  
	sdb     0.00   0.00  0.00   45.00  0.00   224.00   9.96   0.02     0.64  0.00    0.64   0.20   0.90  
	sda     0.00   8.00  0.00    3.00  0.00    44.00  29.33   0.00     0.00  0.00    0.00   0.00   0.00  
	dm-1    0.00   0.00  0.00    2.00  0.00    20.00  20.00   0.00     0.00  0.00    0.00   0.00   0.00  
	dm-7    0.00   0.00  0.00   41.00  0.00   188.00   9.17   0.03     0.83  0.00    0.83   0.22   0.90   
	dm-3    0.00   0.00  0.00    2.00  0.00    16.00  16.00   0.00     0.00  0.00    0.00   0.00   0.00  
	sdd     0.00   0.00  0.00    4.00  0.00    20.00  10.00   0.00     1.00  0.00    1.00   0.50   0.20  

- **rrqm/s**:  read requests placed on the driver request queue and merged per second  
- **wrqm/s**:  write requests placed on the driver request queue and merged per second  
- **r/s**:     read requests issued to the disk device per second  
- **w/s**:     write requests issued to the disk device per second  
- **rkB/s**:   kilobytes read from the disk device per second  
- **wkB/s**: kilobytes written to the disk device per second  
- **avgrq-sz**: average request size in sectors (512 bytes)  
- **avgqu-sz**: average number of requests both waiting in the driver request queue and active on the device  
- **await**: average I/O response time, including time waiting in the driver request queue and the I/O response time of the device (ms)  
- **r_await**: same as await, but for reads only (ms)  
- **w_await**: same as await, but for writes only (ms)  
- **svctm**: average (inferred) I/O response time for the disk device (ms)  
- **%util**: percent of time the device was busy processing I/O requests (utilization)    

####4.2 sar
system activity reporter   

	[root@intel-controller ~(keystone_admin)]# sar -d 1   
	Linux 3.10.0-123.el7.x86_64 (intel-controller)  12/01/2015  _x86_64_(40 CPU)
	
	04:36:24 AM     DEV     tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz await svctm   %util  
	04:36:25 AM    dev8-0  0.00    0.00      0.00      0.00       0.00  0.00    0.00   0.00
	04:36:25 AM   dev8-32  0.00    0.00      0.00      0.00       0.00  0.00    0.00   0.00   
	04:36:25 AM   dev8-48  0.00    0.00      0.00      0.00       0.00  0.00    0.00   0.00   
	04:36:25 AM   dev8-16  2.00    0.00     16.00      8.00       0.01  7.00    7.00   1.40   
	04:36:25 AM  dev253-0  2.00    0.00     16.00      8.00       0.01  7.00    7.00   1.40   
	04:36:25 AM  dev253-1  0.00    0.00      0.00      0.00       0.00  0.00    0.00   0.00   

####4.3 pidstat
   prints CPU usage by default and includes a -d option for disk I/O statistics.    
	
	[root@intel-controller ~(keystone_admin)]# pidstat
	Linux 3.10.0-123.el7.x86_64 (intel-controller)  12/01/2015  _x86_64_(40 CPU)
	
	04:38:43 AM   UID   PID   %usr %system  %guest   %CPU   CPU  Command
	04:38:43 AM    0      1   0.00    0.00    0.00   0.01    17  systemd
	04:38:43 AM    0      2   0.00    0.00    0.00   0.00    13  kthreadd
	04:38:43 AM    0      3   0.00    0.00    0.00   0.00     0  ksoftirqd/0
	04:38:43 AM    0      8   0.00    0.00    0.00   0.00     0  migration/0
	04:38:43 AM    0    250   0.00    0.25    0.00   0.25    12  rcu_sched
	04:38:43 AM    0    251   0.00    0.01    0.00   0.01     2  rcuos/0
	04:38:43 AM    0    252   0.00    0.01    0.00   0.01    14  rcuos/1
	04:38:43 AM    0    253   0.00    0.02    0.00   0.02     6  rcuos/2

####4.4 Dtrace 
don't used ......... *_*

####4.5 SystemTap
####4.6 perf
####4.7 iotop
####4.8 smartctl

	[root@intel-controller ~(keystone_admin)]# smartctl -a  /dev/sdb
	smartctl 6.2 2013-07-26 r3841 [x86_64-linux-3.10.0-123.el7.x86_64] (local build)
	Copyright (C) 2002-13, Bruce Allen, Christian Franke, www.smartmontools.org
	
	=== START OF INFORMATION SECTION ===
	Model Family: HP 500GB SATA disk MM0500EANCR
	Device Model: MM0500EANCR
	Serial Number:9SP11MRZ
	LU WWN Device Id: 5 000c50 015378d79
	Firmware Version: HPG1
	User Capacity:500,107,862,016 bytes [500 GB]
	Sector Size:  512 bytes logical/physical
	Rotation Rate:7200 rpm
	Device is:In smartctl database [for details use: -P show]
	ATA Version is:   ATA8-ACS T13/1699-D revision 6
	SATA Version is:  SATA 2.6, 3.0 Gb/s
	Local Time is:Tue Dec  1 05:22:19 2015 EET
	SMART support is: Available - device has SMART capability.
	SMART support is: Enabled
	
	=== START OF READ SMART DATA SECTION ===
	SMART overall-health self-assessment test result: PASSED
	
	General SMART Values:
	Offline data collection status:  (0x82) Offline data collection activity
	was completed without error.
	Auto Offline Data Collection: Enabled.
	Self-test execution status:  (   0) The previous self-test routine completed
	without error or no self-test has ever
	been run.
	Total time to complete Offline
	data collection:(  659) seconds.
	Offline data collection
	capabilities:(0x7b) SMART execute Offline immediate.
	Auto Offline data collection on/off support.
	Suspend Offline collection upon new
	command.
	Offline surface scan supported.
	Self-test supported.
	Conveyance Self-test supported.
	Selective Self-test supported.
	SMART capabilities:(0x0003) Saves SMART data before entering
	power-saving mode.
	Supports SMART auto save timer.
	Error logging capability:(0x01) Error logging supported.
	General Purpose Logging supported.
	Short self-test routine
	recommended polling time:(   2) minutes.
	Extended self-test routine
	recommended polling time:( 123) minutes.
	Conveyance self-test routine
	recommended polling time:(   3) minutes.
	SCT capabilities:  (0x103f) SCT Status supported.
	SCT Error Recovery Control supported.
	SCT Feature Control supported.
	SCT Data Table supported.
	
	SMART Attributes Data Structure revision number: 10
	Vendor Specific SMART Attributes with Thresholds:
	ID# ATTRIBUTE_NAME           FLAG   VALUE WORST  THRESH    TYPE  UPDATED  WHEN_FAILED RAW_VALUE
	  1 Raw_Read_Error_Rate     0x000f   082   063   044    Pre-fail  Always   -   200649535
	  3 Spin_Up_Time            0x0003   100   100   000    Pre-fail  Always   -   0
	  4 Start_Stop_Count        0x0032   100   100   020    Old_age   Always   -   137
	  5 Reallocated_Sector_Ct   0x0033   100   100   036    Pre-fail  Always   -   10
	  7 Seek_Error_Rate         0x000f   087   060   030    Pre-fail  Always   -   24497902392 
	  9 Power_On_Hours          0x0032   043   043   000    Old_age   Always   -   50364
	 10 Spin_Retry_Count        0x0013   100   100   097    Pre-fail  Always   -   0
	 12 Power_Cycle_Count       0x0032   100   100   020    Old_age   Always   -   138
	180 Unknown_HDD_Attribute   0x003b   100   100   030    Pre-fail  Always   -   1167995224
	184 End-to-End_Error        0x0032   100   100   003    Old_age   Always   -   0
	187 Reported_Uncorrect      0x0032   100   100   000    Old_age   Always   -   0
	188 Command_Timeout         0x0032   100   081   000    Old_age   Always   -   60130469058
	189 High_Fly_Writes         0x003a   087   087   000    Old_age   Always   -   13
	190 Airflow_Temperature_Cel 0x0022   073   047   045    Old_age   Always   -   27 (Min/Max 26/29)
	191 G-Sense_Error_Rate      0x0032   100   100   000    Old_age   Always   -   0
	192 Power-Off_Retract_Count 0x0032   100   100   000    Old_age   Always   -   137
	193 Load_Cycle_Count        0x0032   100   100   000    Old_age   Always   -   138
	194 Temperature_Celsius     0x0022   027   053   000    Old_age   Always   -   27 (0 14 0 0 0)
	195 Hardware_ECC_Recovered  0x001a   030   024   000    Old_age   Always   -   200649535
	196 Reallocated_Event_Count 0x0033   100   100   036    Pre-fail  Always   -   10
	197 Current_Pending_Sector  0x0012   100   100   000    Old_age   Always   -   0
	198 Offline_Uncorrectable   0x0010   100   100   000    Old_age   Offline  -   0
	199 UDMA_CRC_Error_Count    0x003e   200   200   000    Old_age   Always   -   1
	
	SMART Error Log Version: 1
	No Errors Logged
	
	SMART Self-test log structure revision number 1
	No self-tests have been logged.  [To run self-tests, use: smartctl -t]
	
	
	SMART Selective self-test log data structure revision number 1
	 SPAN  MIN_LBA  MAX_LBA  CURRENT_TEST_STATUS
	100  Not_testing
	200  Not_testing
	300  Not_testing
	400  Not_testing
	500  Not_testing
	Selective self-test flags (0x0):
	  After scanning selected spans, do NOT read-scan remainder of disk.
	If Selective self-test is pending on power-up, resume after 0 minute delay.


###5 Experimentation
  using these tools, it’s a good idea to leave iostat(1) continually running so that any result can be immediately double-checked.   

**dd**  

    controller-0:~$ sudo dd if=/dev/sda1 of=/dev/null bs=1024k count=1k
    Password:
    1+0 records in
    1+0 records out
    1048576 bytes (1.0 MB) copied, 0.00225314 s, 465 MB/s

**raw**  可以用来创建字符设备 /dev/raw. 
 
**hdparm**  

    controller-0:~$ sudo hdparm -Tt /dev/sdb
    
    /dev/sdb:
     Timing cached reads:   11902 MB in  2.00 seconds = 5957.06 MB/sec
     Timing buffered disk reads: 1132 MB in  3.00 seconds = 377.06 MB/sec
    
写入时 *avgqu-sz* 和 *await* 会增加   

    Device: rrqm/s   wrqm/s r/s w/srkB/swkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
    sda   0.0022.000.00   15.00 0.00   156.0020.80 0.000.000.000.00   0.00   0.00
    
    Device: rrqm/s   wrqm/s r/s w/srkB/swkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
    sda   0.00 36060.000.00  293.00 0.00 138684.00   946.65 2.137.020.007.02   0.54  15.90
    
    Device: rrqm/s   wrqm/s r/s w/srkB/swkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
    sda   0.00 168874.000.00 1463.00 0.00 666388.00   910.9919.10   12.420.00   12.42   0.59  86.50
    
    Device: rrqm/s   wrqm/s r/s w/srkB/swkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
    sda   0.00 91729.000.00  780.00 0.00 391972.00  1005.06 7.58   11.030.00   11.03   0.64  49.70
    
    Device: rrqm/s   wrqm/s r/s w/srkB/swkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
    sda   0.00 121945.000.00  941.00 0.00 475804.00  1011.27 9.388.710.008.71   0.67  62.60
    
    Device: rrqm/s   wrqm/s r/s w/srkB/swkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
    sda   0.00 145568.000.00 1053.00 0.00 537716.00  1021.3078.17   63.720.00   63.72   0.87  91.80
    
    Device: rrqm/s   wrqm/s r/s w/srkB/swkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
    sda   0.00 98145.000.00  870.00 0.00 395916.00   910.15   140.74  164.110.00  164.11   1.15 100.00
    
    Device: rrqm/s   wrqm/s r/s w/srkB/swkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
    sda   0.00 102841.000.00  827.00 0.00 410860.00   993.62   140.63  169.540.00  169.54   1.21 100.00  
    
###6 Tuning

**ionice**  

- **0, none**: no class specified, so the kernel will pick a default—best effort, with a priority based on the process nice value.  
- **1, real-time**: highest-priority access to the disk. If misused, this can starve other processes (just like the RT CPU scheduling class).  
- **2, best effort**: default scheduling class, supporting priorities 0–7, with 0 the highest.  
- **3, idle**: disk I/O allowed only after a grace period of disk idleness.  

Example operating system tunables    
/sys/block/sda/queue/scheduler (Linux)   

### 7 References
[Patterson 88]
Patterson, D., G. Gibson, and R. Kats. “A Case for Redundant Arrays of Inexpensive Disks.” ACM SIGMOD, 1988.   
[Bovet 05]
Bovet, D., and M. Cesati. Understanding the Linux Kernel, 3rd Edition. O’Reilly, 2005.   
[McDougall 06b]
McDougall, R., J. Mauro, and B. Gregg. Solaris Performance and Tools: DTrace and MDB Techniques for Solaris 10 and OpenSolaris.
Prentice Hall, 2006.   
[Gregg 10b]
Gregg, B. “Visualizing System Latency,” Communications of the ACM, July 2010.   
[Love 10]
Love, R. Linux Kernel Development, 3rd Edition. Addison-Wesley, 2010.   
[Turner 10]
Turner, J. “Effects of Data Center Vibration on Compute System Performance.” USENIX, SustainIT’10.   
[Gregg 11]
Gregg, B., and J. Mauro. DTrace: Dynamic Tracing in Oracle Solaris, Mac OS X and FreeBSD. Prentice Hall, 2011.   
[Cornwell 12]
Cornwell, M. “Anatomy of a Solid-State Drive,” Communications of the ACM, December 2012.   
[Leventhal 13]
Leventhal, A. “A File System All Its Own,” ACM Queue, March 2013.   
[1]
www.youtube.com/watch?v=tDacjrSCeq4   
[2]
http://lwn.net/Articles/332839   
[3]
http://sourceware.org/systemtap/wiki/WSiostatSCSI   
[4]
www.dtracebook.com   
[5]
http://guichaz.free.fr/iotop   
