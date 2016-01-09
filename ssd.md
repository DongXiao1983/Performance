

#How to test SSD

###Testing
To give you a little bit of background when the OSD writes into his journal it uses `D_SYNC` and `O_DIRECT`. Writing with `O_DIRECT` bypasses the Kernel page cache, while `D_SYNC` ensures that the command won’t return until every single write is complete. So yes, basically the OSD forces all the writes to be flushed prior to start the next IO.

First disable the write cache on the disk:

    $ sudo hdparm -W 0 /dev/hda 0
Disable the controller cache, assuming your controller is from HP, in slot 2 and your logical drive is the number 1:


    $ sudo hpacucli ctrl slot=2 modify dwc=disable
    $ sudo hpacucli controller slot=2 logicaldrive 1 modify arrayaccelerator=disable

Now you can start benchmarking your SSD correctly using two different methods. 

###The FIO way:

    
    $ sudo fio --filename=/dev/sda --direct=1 --sync=1 --rw=write --bs=4k --numjobs=1 --iodepth=1 --runtime=60 --time_based --group_reporting --name=journal-test

Now it is important to understand the option we passed:

- **`--filename`**: device we want to test   
- **`--direct`** : we open the device with `O_DIRECT` which means that we are bypassing the Kernel page cache   
- **`--sync`** :  we open the device with `O_DSYNC` we don’t acknowledge until we are sure that the IO has been completely written   
- **`--rw`** : IO pattern, here we use write for sequential writes, journal writes are always sequential    
- **`--bs`** : block size, here we are submitting 4K IOs, this is probably the worst case scenario, so you can always change this value if you know your workload   
- **`--numjobs`** : number of threads that will be running, think this has ceph-osd daemons writing to the journal   
- **`--iodepth`** : we are submitting IO one by one.   
- **`--runtime`** : job duration in seconds   
- **`--time_based`** : run for the specified runtime duration even if the files are completely read or written  
- **`--group_reporting`** : If set, display per-group reports instead of per-job when numjobs is specified.  
- **`--name`** : name of the run  


###II. Ramp up
Increase `--numjobs` through every single new run. Here is a little example on a SSD:   

`--numjobs=1` reports bw 23418KB/s or iops=5854   
`--numjobs=2` reports bw=43697KB/s or iops=10924   
`--numjobs=3` reports bw=63592KB/s or iops=15898   
`--numjobs=4` reports bw=68500KB/s or iops=17124. My SSD is maxing out here   


###III. Interpret the result


###Bonus
If for whatever reasons fio is not available, here is the dd way:

    $ sudo dd if=/dev/urandom of=randfile bs=1M count=1024 && sync
    $ sudo dd if=randfile of=/dev/sda bs=4k count=100000 oflag=direct,dsync


What matters the most here is to find how the SSD is performing while using D_SYNC. At some point users reported some SSD misbehaving with DSYNC. Then you better always test your SSD prior to go in production.   

###Reference:
[ceph-how-to-test-if-your-ssd-is-suitable-as-a-journal-device](http://www.sebastien-han.fr/blog/2014/10/10/ceph-how-to-test-if-your-ssd-is-suitable-as-a-journal-device/ "ceph-how-to-test-if-your-ssd-is-suitable-as-a-journal-device")


[PCIe SSD](http://wenku.it168.com/d_001428570.shtml)
