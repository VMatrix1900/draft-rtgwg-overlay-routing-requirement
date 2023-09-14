---
title: Scenarios and Challenges of Overlay Routing for SD-WAN
abbrev: Overlay Routing for SD-WAN
docname: draft-sheng-rtgwg-overlay-routing-requirement-latest
date:
category: info
submissionType: IETF

ipr: trust200902
area: Routing area
workgroup: rtgwg
keyword: Internet-Draft

coding: us-ascii
pi: [toc, sortrefs, symrefs]

author:
 -
  ins: C. Sheng
  name: Cheng Sheng
  organization: Huawei Technologies
  email: shengcheng@huawei.com
  country: China
 -
  ins: H. Shi
  name: Hang Shi
  organization: Huawei Technologies
  email: shihang9@huawei.com
  country: China
 -
  ins: L. Dunbar
  name: Linda Dunbar
  organization: Futurewei
  email: linda.dunbar@futurewei.com

normative:

informative:

...

--- abstract

Overlay routing is essential during the enterprise networks’ evolution from the interconnection among multiple on-premise branch sites to more advanced ones, such as the interconnection to multi-clouds. This document analyzes the technical requirements and challenges of overlay routing for SD-WAN in these scenarios.

--- middle

# Introduction

SD-WAN is currently widely used in the basic scenarios of one-hop interconnection between enterprise on-premises sites of branches, campuses, and even DCs. With multi-cloud adoption, workloads are migrating to be hosted on clouds. And it is necessary for SD-WAN to interconnect multiple on-premises sites and multiple cloud sites seamlessly with the overlay routing technology.

As the core network technology, overlay routing faces a series of new challenges during its evolution, such as flexible overlay topology formation and auto-provision, global interconnection among multi-regions via multi-ISP networks, and the SLA aware routing across multiple overlay segments. Also, it is necessary to investigate how SD-WAN can be seamlessly integrated with the virtual network of multiple clouds.

# Terminology

- SD-WAN: Software Defined Wide Area Network. In this document, “SD-WAN” refers to policy-driven transporting IP packets over multiple different underlay networks to get better WAN bandwidth management, visibility and control.

- Site: Enterprise sites across different geo regions, where people or workload host, such as branches, campus, or even clouds.

- Edge: The border devices of the SD-WAN site, which could be physical or virtual CPEs.

- TN: Transport Network, the underlay connectivity network which correspond to different ISP network between SD-WAN sites.

- TNP: Transport Network Port, the wan port of the Edge which connect to TN.

- Virtual Tunnel: The IP tunnel between two TNPs of two different edges from different sites.


## Requirements Language

{::boilerplate bcp14-tagged}


# Virtual tunnel auto discovery and provision requirement

As the basis of the SD-WAN overlay network, virtual tunnels between edges should be established before the client routes exchange packets. The virtual tunnels, such as IPSec tunnels, establishment between edges require extensive information exchange, such as public keys, tunnel endpoints properties, etc., which can significantly delay client routes packets forwarding if they are not established ahead of time. A virtual tunnel is a point-to-point forwarding relationship between two SD-WAN Edges across a given or multiple underlay ISP networks that provide a well-defined set of transport characteristics (e.g., delay, security, bandwidth, etc.

~~~
                +------+     +------+
                | Edge |     | Edge |
                +------+     +------+
               /   |    \   /   |    \
              /    |     \ /    |     \
             /     |      X     |      \
            /      |     / \    |       \
           /       |    /   \   |        \
    +------+    +------+     +------+     +------+
    | Edge |    | Edge |     | Edge |     | Edge |
    +------+    +------+     +------+     +------+
~~~
{: #hub-spoke title="Hub spoke topology"}

~~~
                 +---------+
         +-------|   Edge  |---------+
         |       +----+----+         |
         |            |              |
    +---------+       |         +---------+
    |   Edge  |-------+---------|   Edge  |
    +---------+       |         +---------+
         |            |              |
         |            |              |
         |       +----+----+         |
         +-------|   Edge  |---------+
                 +---------+
~~~
{: #full-mesh title="Full mesh topology"}

~~~
    +------+                             +------+
    | Edge |                             | Edge |
    +------+                             +------+
            \                           /
             \                         /
              +------+         +------+
              | Edge |---------| Edge |
              +------+         +------+
             /                         \
            /                           \
    +------+                             +------+
    | Edge |                             | Edge |
    +------+                             +------+
~~~
{: #layered title="Layered topology"}

Different enterprises often have different connectivity topologies with hundreds and thousands of tunnels, as shown in {{hub-spoke}}, {{full-mesh}}, {{layered}}. For the efficiency and simplicity of the O&M, it is highly expected to discover and establish the virtual tunnels between sites automatically instead of manually configuring the overlay tunnels one by one.

{{?I-D.ietf-idr-sdwan-edge-discovery}} has designed an efficient mechanism to exchange the information of each end point of the overlay tunnel by BGP protocol, by which edges could check and decide to establish the tunnel or not automatically. While this mechanism works fine in reality, it can be further improved. For example, it is much more expected to carry more information to reflect the topology intent (Full Mesh, P2MP, P2P) in BGP.

# Topology aware routing with multi hop overlay network requirement

There are many differences in the control plane between the traditional L3 VPN network and the SD-WAN overlay network. As per L3VPN network, IGP protocol (ospf or isis) is deployed on each physical link between routers and is responsible for discovering the underlay network topology and calculating the routing of the BGP nexthops (often loopback0 of PEs), while BGP is deployed to advertise and calculate the VPN routes based on the IGP output. In the SD-WAN overlay network, it is difficult and a not good choice to run IGP directly on the tunnels between edges because it will bring much more resource consumption. p2p tunnels, such as GRE or VXLAN, need to be configured using a virtual interface to run the IGP protocol. Flooding of the IGP message could cause resource waste of the control plane.

For the SD-WAN overlay network, it is recommended to use BGP to discover the overlay topology and calculate the best overlay path, which is also responsible for advertising and calculating the VPN routes.

# SLA aware routing across multiple overlay segments requirement

After a multi-hop SD-WAN overlay network is established, such as the one shown in Figure 4 below, stitching together the overlay tunnels across the Edge1-Edge2-Edge5-Edge6 for the client traffic between Edge1 and Edge6 might provide better SLA than building another other overlay tunnels between Edge1 and Edge6, such as Edge1-Edge2-Edge4-Edge6 and etc. Importing traffic engineering based routing in overlay network can provide more deterministic end-to-end QoS SLA for application.

~~~
                +-------+      +-------+
      + --------| Edge2 |------| Edge4 |-----------+
      |         +-------+      +-------+           |
      |                  \    /                    |
  +-------+               \  /                 +-------+
  | Edge1 |                \/                  | Edge6 |
  +-------+                /\                  +-------+
      |                   /  \                     |
      |                  /    \                    |
      |         +-------+      +-------+           |
      +---------| Edge3 |------| Edge5 |-----------+
                +-------+      +-------+
~~~
{: #SRTE title="Example of SLA aware routing"}

Different application flows have different SLA requirements. For example, voice is sensitive to latency and jitter, while video requires a low packet loss forwarding path. It is necessary to provide some degree of TE function to meet the requirements of different types of applications, which is a new challenge for the SD-WAN overlay networks. Naturally, it is necessary for the centralized SD-WAN controller to collect SLA (latency, packet loss, and bandwidth) information of the tunnels and the overall topology to calculate the segment list satisfying the requirement raised by the application. Further, the data plane that can carry the overlay tunnel list needs to be carefully designed with the consideration of efficiency and productivity.

# Seamless integration with virtual networks of multiple clouds requirement

As more and more enterprises migrate their workloads to multi clouds, it is highly expected to establish a high quality interconnection between the enterprise’s on premise sites and the cloud sites with qualified O&M specification.

It has been widely adopted to create vCPEs on the clouds as cloud edge to bring an uniform experience and O&M method for the access to the clouds. There are also obstacles discovered. For example, how to integrate the multi VPN or multi tenants to the virtual network of different clouds. And for the sake of the reliability, at least two vCPEs need to be created for each cloud site. And it is often recommended to deploy VRRP between the two vCPEs, which need to run VRRP control plane over multicast packets. While for the reason of the security, many cloud service provider closed the native IP multicast services for the tenants. So some new HA feature need to be considered in such scenarios.

Also, different cloud service provider implements different charge rule for the resources of the compute, network etc. It needs to be finely scrutinized to develop the most economical network solution for SD-WAN in cloud networks.

# Overlay multicast over multicast-free underlay networks requirement

As more and more enterprise application are running across SD-WAN overlay networks, multicast traffic is also emerging. Different with traditional multicast VPN networks, SD-WAN overlay networks is based on multiple underlay ISP networks, such as internet, 5G, MPLS etc which does not support multicast.  How to implement an multicast overlay network on top of the multicast-free underlay is challenging. Enhancement to existing SD-WAN routing protocol needs to be made.

# Security Considerations

TBD

# Acknowledgement

The authors would like to thank Haibo Wang, Shunwan Zhuang, Donglei Pang, Hongwei He for their help.
