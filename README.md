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

To [install GNS3](https://docs.gns3.com/docs/getting-started/installation/linux/) under Ubuntu we've got to add a special [ppa](https://itsfoss.com/ppa-guide/) that we could find in the project home page.

I've recollected the options that I use in the configuration of the virtual environment, then I will discuss with you, reader, the most important ones:

- [GNS3 OpenBSD OpenOSPFD](https://photos.app.goo.gl/u5yeCtcYSxE6Yn6Q9)

![vinagre](https://raw.githubusercontent.com/redeltaglio/GNS3-OpenBSD-OpenOSPFD/main/images/gns3_options/5-imp.jpg)

To connect to the virtualized OpenBSD instanced we shall use [vinagre](https://en.wikipedia.org/wiki/Vinagre) with the options specified.

![](https://github.com/redeltaglio/GNS3-OpenBSD-OpenOSPFD/raw/main/images/gns3_options/13-imp.jpg)

Important to underline the virtual interface used by the GNS3 system to nat the connections of the virtual systems to the WAN interface of our workstation. In the case the installation haven't created it we could analyze it with the following commands:

```shell
root@trimurti:/home/riccardo/Work/telecom.lobby/GNS3-OpenBSD-OpenOSPFD# ip link | grep virbr0
4: virbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
7: gns3tap0-0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr0 state UNKNOWN mode DEFAULT group default qlen 1000
root@trimurti:/home/riccardo/Work/telecom.lobby/GNS3-OpenBSD-OpenOSPFD# ip addr | grep virbr0
4: virbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
root@trimurti:/home/riccardo/Work/telecom.lobby/GNS3-OpenBSD-OpenOSPFD# iptables -t nat  -n -v -L
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
 6544 1322K LIBVIRT_PRT  all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain LIBVIRT_PRT (1 references)
 pkts bytes target     prot opt in     out     source               destination         
   27  2636 RETURN     all  --  *      *       192.168.122.0/24     224.0.0.0/24        
    0     0 RETURN     all  --  *      *       192.168.122.0/24     255.255.255.255     
    0     0 MASQUERADE  tcp  --  *      *       192.168.122.0/24    !192.168.122.0/24     masq ports: 1024-65535
   72 14790 MASQUERADE  udp  --  *      *       192.168.122.0/24    !192.168.122.0/24     masq ports: 1024-65535
    0     0 MASQUERADE  all  --  *      *       192.168.122.0/24    !192.168.122.0/24    
root@trimurti:/home/riccardo/Work/telecom.lobby/GNS3-OpenBSD-OpenOSPFD# iptables -t nat -S
-P PREROUTING ACCEPT
-P INPUT ACCEPT
-P OUTPUT ACCEPT
-P POSTROUTING ACCEPT
-N LIBVIRT_PRT
-A POSTROUTING -j LIBVIRT_PRT
-A LIBVIRT_PRT -s 192.168.122.0/24 -d 224.0.0.0/24 -j RETURN
-A LIBVIRT_PRT -s 192.168.122.0/24 -d 255.255.255.255/32 -j RETURN
-A LIBVIRT_PRT -s 192.168.122.0/24 ! -d 192.168.122.0/24 -p tcp -j MASQUERADE --to-ports 1024-65535
-A LIBVIRT_PRT -s 192.168.122.0/24 ! -d 192.168.122.0/24 -p udp -j MASQUERADE --to-ports 1024-65535
-A LIBVIRT_PRT -s 192.168.122.0/24 ! -d 192.168.122.0/24 -j MASQUERADE
root@trimurti:/home/riccardo/Work/telecom.lobby/GNS3-OpenBSD-OpenOSPFD# 
```

If we detect that there's no interface we can create it:

```shell
riccardo@trimurti:~/Work/telecom.lobby/GNS3-OpenBSD-OpenOSPFD$ sudo nmcli con add ifname virbr4 type bridge con-name virbr4
Connection 'virbr4' (4edf28e6-107d-4ae7-baff-b4c65e60604b) successfully added.
riccardo@trimurti:~/Work/telecom.lobby/GNS3-OpenBSD-OpenOSPFD$ nmcli con show | grep virbr4
virbr4           4edf28e6-107d-4ae7-baff-b4c65e60604b  bridge     virbr4 
riccardo@trimurti:~/Work/telecom.lobby/GNS3-OpenBSD-OpenOSPFD$ sudo nmcli connection modify virbr4 ipv4.addresses "192.168.133.1/24"
riccardo@trimurti:~/Work/telecom.lobby/GNS3-OpenBSD-OpenOSPFD$ sudo nmcli connection modify virbr4 ipv4.dns-search "virtual.ama"
riccardo@trimurti:~/Work/telecom.lobby/GNS3-OpenBSD-OpenOSPFD$ sudo nmcli connection modify virbr4 ipv4.method manual
riccardo@trimurti:~/Work/telecom.lobby/GNS3-OpenBSD-OpenOSPFD$ ip addr show dev virbr4
8: virbr4: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether b2:72:e4:84:44:8b brd ff:ff:ff:ff:ff:ff
    inet 192.168.133.1/24 brd 192.168.133.255 scope global noprefixroute virbr4
       valid_lft forever preferred_lft forever
riccardo@trimurti:~/Work/telecom.lobby/GNS3-OpenBSD-OpenOSPFD$ 
```

Using `nmcli` we're launching a shell command that use the [GNOME NetworkManager](https://en.wikipedia.org/wiki/NetworkManager), change will be persistent on reboot. 

![](https://github.com/redeltaglio/GNS3-OpenBSD-OpenOSPFD/raw/main/images/gns3_options/23.jpg)

Another important feature is the [BIOS](https://en.wikipedia.org/wiki/BIOS) binary used by QEMU, we can find it here:

- [PC-BIOS qemu](https://github.com/qemu/qemu/raw/master/pc-bios/bios.bin).

Simply download it.

#### GNS3 network design

![](https://github.com/redeltaglio/GNS3-OpenBSD-OpenOSPFD/raw/main/images/first_layout.jpg)

Using a division by quadrants of the planet earth depending in the geographical position of the VPS routers of our real network  like we're doing in our principle project, here in our virtualized environment we can define three different OSPF areas, concept that is explained by the father of OpenOSPFD, [Claudio Jeker](https://github.com/cjeker), as follow:

> One problem of a link-state protocol is the computation
> cost bourn by every router, particularly in large net-
> works. Many routers have an underpowered CPU and so
> OSPF areas where invented to divide a large network
> into smaller pieces. Every area is connected to a special
> backbone area. In most cases inter-area routing goes via
> the backbone. Routers that are connected to multiple
> areas are area border routers (ABR) and are always con-
> nected to the backbone area. If no direct connection to
> the backbone is possible, a virtual-link has to be estab-
> lished to at least one backbone router. Areas where no
> transit trafﬁc is exchanged can be converted into stub
> areas, reducing the routing table to a bare minimum.
> Stub areas are useful to connect routers with minimal
> memory conﬁgurations to large OSPF clouds.
> LSAs are ﬂooded only inside an area. The ABR has the
> duty to reﬂood the other areas with special summary-
> LSAs to inform them of available preﬁxes inside the
> originating area.

Claudio has been writing a presentation of the daemon that implement the protocol into the OpenBSD kernel and stack:

- [Design and implementation of OpenOSPFD](https://github.com/redeltaglio/GNS3-OpenBSD-OpenOSPFD/raw/main/pdf/OpenOSPFD%20-%20Paper.pdf)

The algorithm is very simple obtaining latitude and longitude from the `egress` IPv4 address, is implemented as usual in [KSH 88 shell](https://web.archive.org/web/20151105130220/http://www2.research.att.com/sw/download/man/man1/ksh88.html) script:

```shell
local=$(ssh -q $vpnc_host.$localdomainname readlink /etc/localtime  | sed "s/\/usr\/share\/zoneinfo\///")
zonetab=$(ssh -q $vpnc_host.$localdomainname cat /usr/share/zoneinfo/zone.tab | grep $local)
[[ $zonetab ]] && echo -e "\n$zonetab\n" || echo -e "\n$local\n"
publicip=$(ssh -q $vpnc_host.$localdomainname ifconfig egress | grep inet |grep -v inet6 | cut -d ' ' -f2)
curl -s "http://ipinfo.io/$publicip" | sed '/readme/d'
loc=$(curl -s "http://ipinfo.io/$publicip" | grep loc | awk '{print $2}' | sed 's/.$//' | sed "s/\"//g")
long=$(echo $loc | cut -d , -f2 | cut -d . -f1)
lat=$(echo $loc | cut -d , -f1 | cut -d . -f1)
echo -e "\nLAT --> $lat"
echo -e "LONG --> $long\n"
if [[ $long -ge -180 && $long -le -60 && $lat -ge 0 ]]; then group=1 && ospfarea="0.0.0.1"; fi
if [[ $long -ge -60 && $long -le 60 && $lat -ge 0 ]]; then group=3 && ospfarea="0.0.0.3"; fi
if [[ $long -ge 60 && $long -le 180 && $lat -ge 0 ]]; then group=5 && ospfarea="0.0.0.5"; fi
if [[ $long -ge -180 && $long -le -60 && $lat -le 0 ]]; then group=2 && ospfarea="0.0.0.2"; fi
if [[ $long -ge -60 && $long -le 60 && $lat -le 0 ]]; then group=4 && ospfarea="0.0.0.4"; fi
if [[ $long -ge 60 && $long -le 180 && $lat -le 0 ]]; then group=6 && ospfarea="0.0.0.6"; fi
echo -e "GROUP --> $group"
echo -e "BACKBONE OSPFAREA 0.0.0.0"
echo -e "GEO OSPFAREA --> $ospfarea\n"
```

 For simplicity now in this virtual environment I've grouped areas into three groups:

- AREA 0.0.1.2 AMERICA
- AREA 0.0.3.4 EUROAFRICA
- AREA 0.0.56 RUCIJAAU that means Asia and Australia basically.

#### Basic router installation

Let's introduce the most minimal installation despite space allocation of our hard disk that is a [NVMe SSD](https://en.wikipedia.org/wiki/NVM_Express) to archive the best performance possible. There is many at low prices available onto web shopping pages.

Look at the video!

​	[![GNS3 OpenBSD OpenOSPFD](http://img.youtube.com/vi/EzcM3zPmjfE/0.jpg)](https://youtu.be/EzcM3zPmjfE "GNS3 OpenBSD OpenOSPFD")

Meanwhile I want to specify to you, reader, that in this installation process I full allocate a [qcow2](https://en.wikipedia.org/wiki/Qcow) disk of 900 mega, installing only those [file sets](https://www.openbsd.org/faq/faq4.html#FilesNeeded):

- `bsd.mp`
- `base69.tgz`

#### OSPF tools under OpenBSD operative system

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e4/DijkstraDemo.gif/220px-DijkstraDemo.gif)

*[Dijkstra algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm), the father of OSPF.*



#### Type one, clean ethernet and OSPF

