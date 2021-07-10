# OSPF OpenBSD brief, a virtualization environment
![Go ahead explore!](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSaOGhYpqhVx6p-mJd1e_foOrVHiEAWHRPjNA&usqp=CAU)

*A brief about the OpenBSD implementation of the OSPF protocol. All under GNS3 virtualization environment.*

OpenOSPFD is part of the [OpenBGPD](http://www.openbgpd.org/) project to implement in the OpenBSD kernel various routing protocols, [interior](https://en.wikipedia.org/wiki/Interior_gateway_protocol) and [exterior](https://en.wikipedia.org/wiki/Exterior_Gateway_Protocol).

In particular [OSPF](https://en.wikipedia.org/wiki/Open_Shortest_Path_First) pertains to the IGP [link-state routing protocols](https://en.wikipedia.org/wiki/Link-state_routing_protocol).

Our goal is to port a network design from an optimum article found on the web to the OpenBSD platform. The article is:

- [Overloading a whole bunch of different OSPF implmentations with a shitload of External LSAs](https://www.blackhole-networks.com/OSPF_overload/) 

An this is the network design that we will implement under a [GNS3](https://en.wikipedia.org/wiki/Graphical_Network_Simulator-3) virtual environment running into [KVM](https://en.wikipedia.org/wiki/Kernel-based_Virtual_Machine):

![](https://github.com/redeltaglio/GNS3-OpenBSD-OpenOSPFD/raw/main/images/210184184_10226903997608520_8849131688279375575_n.jpg)

Obviously that design is used to study and understand **how does it works**! Next we will apply it to our [OpenBSD guerrilla MESH network](https://github.com/redeltaglio/OpenBSD)!

#### Starting with GNS3

![](https://docs.gns3.com/img/logocolour.png)

GNS3 is a network software emulator that run under the Linux kernel. In out workstation we've got an Ubuntu operative system with those parameters:

```shell
riccardo@trimurti:~/Work/telecom.lobby/GNS3-OpenBSD-OpenOSPFD$ uname -a
Linux trimurti 5.11.0-22-generic #23-Ubuntu SMP Thu Jun 17 00:34:23 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
riccardo@trimurti:~/Work/telecom.lobby/GNS3-OpenBSD-OpenOSPFD$ cat /etc/apt/sources.list | grep -v \# | grep security
deb http://security.ubuntu.com/ubuntu hirsute-security main restricted
deb http://security.ubuntu.com/ubuntu hirsute-security universe
deb http://security.ubuntu.com/ubuntu hirsute-security multiverse
riccardo@trimurti:~/Work/telecom.lobby/GNS3-OpenBSD-OpenOSPFD$ lscpu
Architecture:                    x86_64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
Address sizes:                   39 bits physical, 48 bits virtual
CPU(s):                          8
On-line CPU(s) list:             0-7
Thread(s) per core:              2
Core(s) per socket:              4
Socket(s):                       1
NUMA node(s):                    1
Vendor ID:                       GenuineIntel
CPU family:                      6
Model:                           94
Model name:                      Intel(R) Core(TM) i7-6700K CPU @ 4.00GHz
Stepping:                        3
CPU MHz:                         1387.328
CPU max MHz:                     4700,0000
CPU min MHz:                     800,0000
BogoMIPS:                        7999.96
Virtualization:                  VT-x
L1d cache:                       128 KiB
L1i cache:                       128 KiB
L2 cache:                        1 MiB
L3 cache:                        8 MiB
NUMA node0 CPU(s):               0-7
Vulnerability Itlb multihit:     KVM: Mitigation: VMX disabled
Vulnerability L1tf:              Mitigation; PTE Inversion; VMX conditional cache flushes, SMT vulnerable
Vulnerability Mds:               Mitigation; Clear CPU buffers; SMT vulnerable
Vulnerability Meltdown:          Mitigation; PTI
Vulnerability Spec store bypass: Mitigation; Speculative Store Bypass disabled via prctl and seccomp
Vulnerability Spectre v1:        Mitigation; usercopy/swapgs barriers and __user pointer sanitization
Vulnerability Spectre v2:        Mitigation; Full generic retpoline, IBPB conditional, IBRS_FW, STIBP conditional, RSB filling
Vulnerability Srbds:             Mitigation; Microcode
Vulnerability Tsx async abort:   Mitigation; Clear CPU buffers; SMT vulnerable
Flags:                           fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf 
                                 pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ssbd ibrs ibpb sti
                                 bp tpr_shadow vnmi flexpriority ept vpid ept_ad fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp
                                  md_clear flush_l1d
riccardo@trimurti:~/Work/telecom.lobby/GNS3-OpenBSD-OpenOSPFD$ free -m
              total        used        free      shared  buff/cache   available
Mem:          23951        6511       15181        1098        2258       16151
Swap:          2047           0        2047
riccardo@trimurti:~/Work/telecom.lobby/GNS3-OpenBSD-OpenOSPFD$ 
```

*Nothing special at all!*

