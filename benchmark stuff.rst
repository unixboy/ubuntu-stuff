Here's a direct read.

       $ sudo hdparm -t /dev/sda2

/dev/sda2:
 Timing buffered disk reads: 302 MB in  3.00 seconds = 100.58 MB/sec

And here's a cached read.

       $ sudo hdparm -T /dev/sda2

/dev/sda2:
 Timing cached reads:   4636 MB in  2.00 seconds = 2318.89 MB/sec
 Details

-t     Perform  timings  of  device reads for benchmark and comparison 
       purposes.  For meaningful results, this operation should be repeated
       2-3 times on an otherwise inactive system (no other active processes) 
       with at least a couple of megabytes of free memory.  This displays  
       the  speed of reading through the buffer cache to the disk without 
       any prior caching of data.  This measurement is an indication of how 
       fast the drive can sustain sequential data reads under Linux, without 
       any filesystem overhead.  To ensure accurate  measurements, the 
       buffer cache is flushed during the processing of -t using the 
       BLKFLSBUF ioctl.

-T     Perform timings of cache reads for benchmark and comparison purposes.
       For meaningful results, this operation should be repeated 2-3
       times on an otherwise inactive system (no other active processes) 
       with at least a couple of megabytes of free memory.  This displays
       the speed of reading directly from the Linux buffer cache without 
       disk access.  This measurement is essentially an indication of the
       throughput of the processor, cache, and memory of the system under 
       test.

Using dd

I too have used dd for this type of testing as well. One modification I would make to the above command is to add this bit to the end of your command, ; rm ddfile.

$ time sh -c "dd if=/dev/zero of=ddfile bs=8k count=250000 && sync"; rm ddfile
 
