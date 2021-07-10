# OSPF OpenBSD brief, a virtualization environment
![Go ahead explore!](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSaOGhYpqhVx6p-mJd1e_foOrVHiEAWHRPjNA&usqp=CAU)

*A brief about the OpenBSD implementation of the OSPF protocol. All under GNS3 virtualization environment.*

OpenOSFD is part of the [OpenBGPD](http://www.openbgpd.org/) project to implement in the OpenBSD kernel various routing protocols, [interior](https://en.wikipedia.org/wiki/Interior_gateway_protocol) and [exterior](https://en.wikipedia.org/wiki/Exterior_Gateway_Protocol).

In particular [OSPF](https://en.wikipedia.org/wiki/Open_Shortest_Path_First) pertains to the IGP [link-state routing protocols](https://en.wikipedia.org/wiki/Link-state_routing_protocol).

Our goal is to port a network design from an optimum article found on the web to the OpenBSD platform. The article is:

- [Overloading a whole bunch of different OSPF implmentations with a shitload of External LSAs](https://www.blackhole-networks.com/OSPF_overload/) 

An this is the network design that we will implement under a [GNS3](https://en.wikipedia.org/wiki/Graphical_Network_Simulator-3) virtual environment running into [KVM](https://en.wikipedia.org/wiki/Kernel-based_Virtual_Machine):

![](https://github.com/redeltaglio/GNS3-OpenBSD-OpenOSPFD/raw/main/images/210184184_10226903997608520_8849131688279375575_n.jpg)

Obviously that design is used to study and understand **how does it works**! Next we will apply it to our [OpenBSD guerrilla MESH network](https://github.com/redeltaglio/OpenBSD)!

