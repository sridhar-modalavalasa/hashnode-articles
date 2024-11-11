---
title: "Pod to Pod Communication Across The Nodes"
datePublished: Mon Nov 11 2024 05:57:17 GMT+0000 (Coordinated Universal Time)
cuid: cm3cm2w2l000809l5fh6f6hqd
slug: pod-to-pod-communication-across-the-nodes
tags: pod-pod-communcation

---

Here’s a breakdown of the flow of the packet from Pod1 in VM1 to Pod2 in VM2, including key steps and networking concepts involved:

### Packet Transmission from Pod1 to the Root Namespace (VM1):

* The packet originates from **Pod1** in **VM1** and is sent via its Ethernet device.
    
* Inside VM1, Pod1’s Ethernet device is paired with a **virtual Ethernet (veth) device** in the root namespace.
    
* The packet travels through this veth pair, entering the **root namespace** (host namespace) of VM1.
    

### Processing in the Root Namespace of VM1:

* Once in the root namespace of VM1, the packet arrives at a **network bridge** (commonly a Linux bridge).
    
* The bridge, which links the network interfaces of multiple pods, attempts to resolve the destination MAC address using **ARP (Address Resolution Protocol)**.
    
* Since there is no connected device on this bridge with the corresponding MAC address, the ARP lookup fails.
    
* As a result, the bridge forwards the packet to the **default route**, sending it out via **eth0** (the main network interface of VM1).
    

### Packet Routing through the Network:

* The packet leaves **VM1** and enters the **external network**.
    
* The network layer (usually an SDN or a cloud provider’s network layer) then routes the packet based on the **CIDR block** assigned to the destination Node (VM2).
    
* Using the destination CIDR, the network routes the packet to the **correct Node**, VM2.
    

### Arrival at the Root Namespace of VM2:

* The packet arrives at **eth0** in the root namespace of VM2.
    
* From there, the network stack determines that the packet is destined for **Pod2**.
    

### Final Delivery to Pod2:

* The packet is routed from the root namespace of VM2 through the **virtual Ethernet (veth) pair** leading to **Pod2’s namespace**.
    
* It finally arrives at **Pod2’s network interface**, completing the transmission from Pod1 in VM1 to Pod2 in VM2.