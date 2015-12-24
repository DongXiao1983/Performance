###BIOS设置

###Hyper-Threading(HT)
基本做云平台的，VT和HT打开都是必须的，超线程技术(HT)就是利用特殊的硬件指令，把两个逻辑内核模拟成两个物理芯片，让单个处理器都能使用线程级并行计算，进而兼容多线程操作系统和软件，减少了CPU的闲置时间，提高的CPU的运行效率。   

###关闭节能
关闭节能后，对性能还是有所提升的，所以坚决调整成性能型(Performance)。当然也可以在操作系统级别进行调整，详细的调整过程请参考链接，但是不知道是不是由于BIOS已经调整的缘故，所以在CentOS 6.6上并没有发现相关的设置。   


    for CPUFREQ in /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor; do [ -f $CPUFREQ ] || continue; echo -n performance > $CPUFREQ; done  

###NUMA
简单来说，NUMA思路就是将内存和CPU分割为多个区域，每个区域叫做NODE,然后将NODE高速互联。 node内cpu与内存访问速度快于访问其他node的内存，NUMA可能会在某些情况下影响ceph-osd。解决的方案，一种是通过BIOS关闭NUMA，另外一种就是通过cgroup将ceph-osd进程与某一个CPU Core以及同一NODE下的内存进行绑定。但是第二种看起来更麻烦，所以一般部署的时候可以在系统层面关闭NUMA。CentOS系统下，通过修改`/etc/grub.conf`文件，添加`numa=off`来关闭NUMA。   

    kernel /vmlinuz-2.6.32-504.12.2.el6.x86_64 ro root=UUID=870d47f8-0357-4a32-909f-74173a9f0633 rd_NO_LUKS rd_NO_LVM LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM   biosdevname=0 numa=off  


###Kernel pid max
    echo 4194303 > /proc/sys/kernel/pid_max

###Jumbo frame
 交换机端需要支持该功能，系统网卡设置才有效果

    ifconfig eth0 mtu 9000
    echo "MTU=9000" | tee -a /etc/sysconfig/network-script/ifcfg-eth0
    /etc/init.d/networking restart  

###read_ahead
 通过数据预读并且记载到随机访问内存方式提高磁盘读操作，查看默认值

    cat /sys/block/sda/queue/read_ahead_kb


    echo "8192" > /sys/block/sda/queue/read_ahead_kb  

###swappiness
 主要控制系统对swap的使用，这个参数的调整最先见于UnitedStack公开的文档中，猜测调整的原因主要是使用swap会影响系统的性能。

    echo "vm.swappiness = 0" | tee -a /etc/sysctl.conf  

###I/O Scheduler
简单说SSD要用noop，SATA/SAS使用deadline。

    echo "deadline" > /sys/block/sd[x]/queue/scheduler
    echo "noop" > /sys/block/sd[x]/queue/scheduler  

###Others
cgroup    
不在process和thread在不同的core上移动(更好的缓存利用)   
减少NUMA的影响   
网络和存储控制器影响 - 较小   
通过限制cpuset来限制Linux调度域(不确定是不是重要但是是最佳实践)   
如果开启了HT，可能会造成OSD在thread1上，KVM在thread2上，并且是同一个core。Core的延迟和性能取决于其他一个线程做什么。   


###Reference
[Ceph性能优化总结(v0.94)](http://xiaoquqi.github.io/blog/2015/06/28/ceph-performance-optimization-summary/)
