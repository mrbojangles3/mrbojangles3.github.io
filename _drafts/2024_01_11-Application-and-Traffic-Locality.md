---
title: "Traffic and Application Locality"
categories:
  - Blog
tags:
  - System Administration
  - Networking
  - Receive Side Steering (RSS)
  - taskset
  - Locality
---

# Overview
Most of this document will reference sections of [sections of the Linux kernel docs](https://www.kernel.org/doc/html/latest/networking/scaling.html). I wil use iperf3 server as test application to illustrate the differences between aligning all of your interupts, hashes, and application work.

We need a way to watch the goodness that can come from locality. For that we will use `perf`. With perf we can watch CPU performance counters 
## Test Application
iperf3 server connected via lan to the iperf3 client. iperf3 server will be launched with tasksel to select the appropriate cores. Lets add a separate IP address to this interface so we don't have all the host traffic interfering with our test. `ip addr add 192.168.3.20/24 dev enp12s0`.

## Test Hardware
NIC features vary widely so its important to document the hardware and how you are configuring it. I am using an [Intel i211](https://www.intel.com/content/www/us/en/content-details/333017/intel-ethernet-controller-i211-datasheet.html) which uses the `igb` driver. 
```
ethtool -i enp12s0
driver: igb
version: 6.6.9-100.fc38.x86_64
firmware-version:  0. 6-1
expansion-rom-version: 
bus-info: 0000:0c:00.0
supports-statistics: yes
supports-test: yes
supports-eeprom-access: yes
supports-register-dump: yes
supports-priv-flags: yes
```
The CPU is an 8 core Intel Skylake i7-6700. This system has 4 cores but 8 "threads". 

To get the ability to configure ntuples, turn it on, `sudo ethtool --features enp12s0 ntuple on`

## Steering traffic to a core
According to the Linux kernel docs above there is no
### RPS
### RFS


## Baseline Performance
TODO: What is the best way to show the efficiency
From the client: `iperf3 -c 192.168.5.2 -t 30`
For a 30 second iperf3 run we see the following from `perf stat`
```
 Performance counter stats for 'iperf3 -s -B 192.168.5.2':

            736.86 msec task-clock:u                     #    0.034 CPUs utilized             
                 0      context-switches:u               #    0.000 /sec                      
                 0      cpu-migrations:u                 #    0.000 /sec                      
               228      page-faults:u                    #  309.421 /sec                      
        65,354,007      cycles:u                         #    0.089 GHz                       
        20,718,094      instructions:u                   #    0.32  insn per cycle            
         4,534,429      branches:u                       #    6.154 M/sec                     
            51,534      branch-misses:u                  #    1.14% of all branches           

      21.990470393 seconds time elapsed

       0.059392000 seconds user
       0.471623000 seconds sys
```

## Keeping the application on the same core

## Dedicating the core to just this work
