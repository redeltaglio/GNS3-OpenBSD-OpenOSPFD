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

Look at the video, using maximum resolution!

​	[![GNS3 OpenBSD OpenOSPFD](http://img.youtube.com/vi/EzcM3zPmjfE/0.jpg)](https://youtu.be/EzcM3zPmjfE "GNS3 OpenBSD OpenOSPFD")

Meanwhile I want to specify to you, reader, that in this installation process I full allocate a [qcow2](https://en.wikipedia.org/wiki/Qcow) disk of 900 mega, installing only those [file sets](https://www.openbsd.org/faq/faq4.html#FilesNeeded):

- `bsd.mp`
- `base69.tgz`

```shell
riccardo@trimurti:~/GNS3/projects/openospfd$ find .
.
./openospfd.gns3
./project-files
./project-files/qemu
./project-files/qemu/91de3fa3-3998-48d9-ba78-275f0b2689ec
./project-files/qemu/91de3fa3-3998-48d9-ba78-275f0b2689ec/hdd_disk.qcow2
./project-files/qemu/91de3fa3-3998-48d9-ba78-275f0b2689ec/hda_disk.qcow2
./project-files/qemu/91de3fa3-3998-48d9-ba78-275f0b2689ec/qemu-img.log
./project-files/qemu/91de3fa3-3998-48d9-ba78-275f0b2689ec/qemu.log
riccardo@trimurti:~/GNS3/projects/openospfd$ 
```

As you can appreciate there are two important informations in the files created by the software:

- `91de3fa3-3998-48d9-ba78-275f0b2689ec`: the id of the `openbsd-1` virtual router created.
- `hda_disk.qcow2`: the virtual hard disk of the system

Now we know that this virtual hd is a clear minimal router installation and the only difference by one router to another is the `hostname`. The mac addresses are created by the application. Analyze it with `qemu-img` tool:

```shell
riccardo@trimurti:~/GNS3/projects/openospfd/project-files/qemu/91de3fa3-3998-48d9-ba78-275f0b2689ec$ qemu-img info hda_disk.qcow2 
image: hda_disk.qcow2
file format: qcow2
virtual size: 900 MiB (943718400 bytes)
disk size: 484 MiB
cluster_size: 65536
backing file: /home/riccardo/GNS3/images/QEMU/openbsd-hda.qcow2
Format specific information:
    compat: 1.1
    compression type: zlib
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
    extended l2: false
riccardo@trimurti:~/GNS3/projects/openospfd/project-files/qemu/91de3fa3-3998-48d9-ba78-275f0b2689ec$
```

As you can see GNS3 trade it like a backing file of another. What we want to archive is that the minimal install is traded as the default disk image. So we can transfer changes to the primary:

```shell
riccardo@trimurti:~/GNS3/projects/openospfd/project-files/qemu/91de3fa3-3998-48d9-ba78-275f0b2689ec$ qemu-img commit hda_disk.qcow2 
Image committed.
riccardo@trimurti:~/GNS3/projects/openospfd/project-files/qemu/91de3fa3-3998-48d9-ba78-275f0b2689ec$ 
```

One time we've done this simple step we can go ahead building the layout that I have presented before.

[![GNS3 OpenBSD OpenOSPFD part 2](http://img.youtube.com/vi/Xwggn75pqKw/0.jpg)](https://youtu.be/Xwggn75pqKw "GNS3 OpenBSD OpenOSPFD part 2")

What I've changed from the first installation? [Elementary my dear Watson](https://www.youtube.com/watch?v=lag22Hl2RQw):

```shell
# cat > /etc/myname
2.virtual.ama
^D
# rm -rf /etc/ssh/ssh_host_*
```

#### Type one, clean ethernet and OSPF

![](https://github.com/redeltaglio/GNS3-OpenBSD-OpenOSPFD/raw/main/images/clean_ethernet_layout.png)

Let's start a simple layout that will see the OSPF protocol running upon four ethernet network and nothing more. As you can see labels tell us what interface is connected with. Remember that we've not installed the man file set, better saying that we can consult it using another machine. I've got many in my MESH OpenBSD IPSec network!

In the next two videos the configuration first of the interfaces members of the `area 0.0.3.4` next the others twos and at least but not last the `backbone` area. 

[![GNS3 OpenBSD OpenOSPFD part 3](http://img.youtube.com/vi/OoZxB8zEZ8E/0.jpg)](https://youtu.be/OoZxB8zEZ8E "GNS3 OpenBSD OpenOSPFD part 3")

[![GNS3 OpenBSD OpenOSPFD part 4](http://img.youtube.com/vi/Rpnsy0FV6WU/0.jpg)](https://youtu.be/Rpnsy0FV6WU "GNS3 OpenBSD OpenOSPFD part 4")

#### OSPF tools under OpenBSD operative system

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e4/DijkstraDemo.gif/220px-DijkstraDemo.gif)

*[Dijkstra algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm), the father of OSPF.*

Let's start configure in a very basic form OpenOSPFD in our OpenBSD routers. For now we will configure the four areas and let the interfaces become part of. We will not touch default values of the daemon. Remember that we've got to configure a unique `router-id` for every one.

See this simple video, (with some errors dude I'm not perfect!), it's about the clearest possible configuration of router 1, 2 and 5:

[![GNS3 OpenBSD OpenOSPFD part 5](http://img.youtube.com/vi/QjRulRUN-l8/0.jpg)](https://youtu.be/QjRulRUN-l8 "GNS3 OpenBSD OpenOSPFD part 5")

Following the article "[OSPF Deep Dive](https://www.blackhole-networks.com/OSPF/)" we are going to understand some bases and some command about the IGP protocol under the OpenBSD environment.

So we remind to you, reader, that in effect it is an IGP that route IP within a single [routing domain](https://en.wikipedia.org/wiki/Routing_domain), in my case in development it is the virtualized layout of GNS3 with local domain name `virtual.ama` and in production the OpenBSD VPS hosts interconnected by IPSec that reply to the local domain name `telecom.lobby`.  Each router gathers link state information and build a complete network topology for the entire routing domain. From the evolution necessary to the [RIP protocol](https://en.wikipedia.org/wiki/Routing_Information_Protocol) two types of IGP was developed:

- [Distance vector](https://en.wikipedia.org/wiki/Distance-vector_routing_protocol)
- [Link State](https://en.wikipedia.org/wiki/Link-state_routing_protocol)

OSPF is defined in various [RFC](https://en.wikipedia.org/wiki/Request_for_Comments):

- [RFC 2328](http://tools.ietf.org/html/rfc2328)
- [RFC 3101](https://datatracker.ietf.org/doc/html/rfc3101)
- [RFC 3630](https://datatracker.ietf.org/doc/html/rfc3630)
- [RFC 3623](https://datatracker.ietf.org/doc/html/rfc3623)

OSPF runs on top of IP using `protocol 89` for IPv4. Use two [multicast](https://en.wikipedia.org/wiki/Multicast) address `224.0.0.5` `AllSPFRouters` and `224.0.0.6` `AllDRRouters`, all routers discover each other with a hello mechanism sent to `AllSPFRouters` in the same broadcast domain obviously that when is received provoke comparison of configuration parameters that is agree form an adjacency and become neighbors then they advertise themselves and their connectivity in Link State Advertisements `LSAs` to their neighbors. All `LSAs` received minus where they received from is flooded to the others neighbors, with a reliable flooding mechanism. Every `LSA` must be acknowledged through a specif package or by seeing the `LSAs ID` that was sent in another update from neighbor the `LSA` was sent to. `LSA ACK` can be send direct by an unicast or delayed if we've got received some and ack with a single. `LSA` got a timer and when it pass they are aged and must be periodically refreshed, routers use `LSA` to construct the Link State Database `LSDB`, that contains topology information for the entire routing domain, used to calculate paths to every other node using Dijkstra's Shortest Path Algorithm. The routing domain into smaller regions known as areas, and can summarize routing information between areas to cut down on the size of the `LSDB` and the routing tables , each area has it's own `LSDB`. Makes use of a two level hierarchy creating a simple [hub and spoke topology](https://networklessons.com/tag/hub-and-spoke). Top level of the hierarchy is known as `backbone` area `0.0.0.0`. All areas are connected directly to the backbone. There are mechanisms to allow tunneling of routing information to get around this restriction.

So `LSAs` are recollected into a database known as `LSDB` that is a model of the topology of the network. It contains tuples of the node's router id, neighbors router id, and a cost to that neighbor; the router runs SPF algorithm on the `LSDB` every the `LSDB` changes.

`LSDB` can be viewed under OpenBSD with those commands:

```shell
8# ospfctl show database summary

                Summary Net Link States (Area 0.0.5.6)

LS age: 149
Options: -|-|-|-|-|-|E|-
LS Type: Summary (Network)
Link State ID: 192.168.0.0 (Network ID)
Advertising Router: 1.1.1.7
LS Seq Number: 0x80000001
Checksum: 0xdb04
Length: 28
LS age: 149
Options: -|-|-|-|-|-|E|-
LS Type: Summary (Network)
Link State ID: 192.168.1.0 (Network ID)
Advertising Router: 1.1.1.7
LS Seq Number: 0x80000001
Checksum: 0x359f
Length: 28
LS age: 149
Options: -|-|-|-|-|-|E|-
LS Type: Summary (Network)
Link State ID: 192.168.3.0 (Network ID)
Advertising Router: 1.1.1.7
LS Seq Number: 0x80000001
Checksum: 0x1fb3
Length: 28
8# 

8# ospfctl show database  

                Router Link States (Area 0.0.5.6)

Link ID         Adv Router      Age  Seq#       Checksum
1.1.1.7         1.1.1.7         24   0x80000003 0x143d
1.1.1.8         1.1.1.8         23   0x80000002 0x054d

                Net Link States (Area 0.0.5.6)

Link ID         Adv Router      Age  Seq#       Checksum
192.168.5.2     1.1.1.8         23   0x80000001 0x2f9a

                Summary Net Link States (Area 0.0.5.6)

Link ID         Adv Router      Age  Seq#       Checksum
192.168.0.0     1.1.1.7         48   0x80000001 0xdb04
192.168.1.0     1.1.1.7         48   0x80000001 0x359f
192.168.3.0     1.1.1.7         48   0x80000001 0x1fb3

                Summary Router Link States (Area 0.0.5.6)

Link ID         Adv Router      Age  Seq#       Checksum
1.1.1.1         1.1.1.7         48   0x80000001 0x51f2

                Type-5 AS External Link States

Link ID         Adv Router      Age  Seq#       Checksum
0.0.0.0         1.1.1.8         69   0x80000001 0x9769
8# 


```

To show interfaces that participate to the OSPF area:

```shell
8# ospfctl show interfaces                                                                 
Interface   Address            State  HelloTimer Linkstate  Uptime    nc  ac
vio1        192.168.122.25/24  DOWN   -          active     00:00:00   0   0
vio0        192.168.5.2/24     DR     00:00:06   active     00:07:59   1   1
8# 
```

Show FIB and RIB, remembering that:

> The forwarding information base (FIB) is the actual information that a routing/switching device uses to choose the interface that a given packet will use for egress. For example, the FIB might be programmed such that a packet bound to a destination in 192.168.1.0/24 should be sent out of physical port ethernet1/2. There may actually be multiple FIB's on a device for unicast forwarding vs multicast RPF checking, different protocols (ip vs mpls vs ipv6) but the basic function is the same - selection criteria (usually destination) mapping to output interface/encapsulation. Individual FIB's may also be partitioned to achieve concurrent independent forwarding tables (i.e. vrf's).
>
> Each FIB is programmed by one or more routing information bases (RIB). The RIB is a selection of routing information learned via static definition or a dynamic routing protocol. The algorithms used within various RIB's will vary - so, for example, the means by which BGP or OSPF determines potential best paths vary quite a bit. The means by which multiple RIB's are programmed into a common (set) of FIB's in a box will vary by implementation but this is where concepts like administrative distance are used (e.g. identical paths are learned via eBGP and OSPF, the eBGP is usually preferred for FIB injection). Again, RIB's may also be potentially partitioned to allow for multiple vrf's, etc.

```shell
8# ospfctl show fib                                                                                                                                                                 
flags: * = valid, O = OSPF, C = Connected, S = Static
Flags  Prio Destination          Nexthop          
*S        8 0.0.0.0/0            192.168.122.1
*C        4 1.1.1.8/32           1.1.1.8
*C        0 127.0.0.0/8          link#0
*S        8 127.0.0.0/8          127.0.0.1
*         1 127.0.0.1/32         127.0.0.1
*O       32 192.168.0.0/24       192.168.5.1
*O       32 192.168.1.0/24       192.168.5.1
*O       32 192.168.3.0/24       192.168.5.1
*C        4 192.168.5.0/24       192.168.5.2
*C        4 192.168.122.0/24     192.168.122.25
*S        8 224.0.0.0/4          127.0.0.1
8# ospfctl show rib 
Destination          Nexthop           Path Type    Type      Cost    Uptime  
1.1.1.1              192.168.5.1       Inter-Area   Router    20      00:31:08
1.1.1.7              192.168.5.1       Intra-Area   Router    10      00:31:08
1.1.1.8              0.0.0.0         C Intra-Area   Router    0       00:31:59
192.168.0.0/24       192.168.5.1       Inter-Area   Network   20      00:31:08
192.168.1.0/24       192.168.5.1       Inter-Area   Network   30      00:31:08
192.168.3.0/24       192.168.5.1       Inter-Area   Network   30      00:31:08
192.168.5.0/24       192.168.5.2     C Intra-Area   Network   10      00:31:13
192.168.122.0/24     0.0.0.0         C Intra-Area   Network   10      00:31:59
8# 
```

#### OpenBSD openospfd configuration

![](https://github.com/redeltaglio/GNS3-OpenBSD-OpenOSPFD/raw/main/images/WAN.png)

*OpenOSPF configuration:*

```shell
8# cat ospfd.conf                                                              
router-id 1.1.1.8

area 0.0.5.6 {
	interface vio0
	interface vio1 {passive}
}
8# 

7# cat ospfd.conf                                                              

router-id "1.1.1.7"

area 0.0.0.0 {
	interface vio1
}
area 0.0.5.6 {
	interface vio0
}
7# 
1# cat ospfd.conf                                                              
router-id "1.1.1.1"

area 0.0.0.0 {
	interface vio1

}
area 0.0.3.4 {
	interface vio0
}
1#
3# cat ospfd.conf                                                              
router-id "1.1.1.3"

area 0.0.3.4 {
	interface vio0
}
3# 
5# cat ospfd.conf                                                              
router-id "1.1.1.5"

area 0.0.0.0 {
	interface vio0
}

area 0.0.1.2 {
	interface vio1
}
5# 
```

