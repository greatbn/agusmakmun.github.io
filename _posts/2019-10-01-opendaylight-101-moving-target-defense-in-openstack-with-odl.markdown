---
layout: post
title: Opendaylight 101 - Moving target defense in Openstack with ODL
date: '2019-10-01 02:49:49'
---

In this post, I will talk about moving target defense in SDN-based cloud. I will use the previous system which are Openstack and Opendaylight for SDN-based cloud environment.

Firstly, I brief introduce about Moving target defense (MTD). This is the security mechanism to protect the system, services. MTD was proposed at first in the U.S national cyber leap year summit in 2009. The main idea of MTD is try to shift and change over time properties of victim to increase complexity and cost for attackers. It also limits the exposure of vulnerabilities and opportunities for attack and increase system resilency.

In SDN-based Cloud, the virtual network are controlled by SDN Controller. It's easier to perform MTD than the traditional system. You can see in figure 1.

<img src="https://imgur.com/NacpMKZ.png">
*Figure 1: MTD in SDN-based Cloud*

When attack try to attack red VM, but the yellow VM will answer the request from attack. The attacker does not know about this scenario because when the packet goes out from yellow VM, the source IP address of the packet is changed to IP address of red VM. 

The SDN switch are installed redirect rules, include forward rule and reverse rule [1]. But when we integrate Opendaylight with Openstack, the Netvirt creates alot of flow tables. The table are visualize in figure 2 [2].

<img src="https://imgur.com/FacDI0R.png">
*Figure 2: Flow table of Opendaylight - Netvirt pipeline*

Based on the function of each table, we will install forward link rules in table 27 and reverse link in table 28.

Table 27 is a `DNAT (FIP)`, and this table will handle packets from outside with external destination IP and change to internal IP address. In our case, we will change the destination IP to other VM if it matches the others criteria such as source IP address or source Port.

Table 28 is a `SNAT (FIP)`, and this table will handle packets from inside to outside with internal destination IP and change to external IP address. We have to change the source IP address to the victim IP address to fool the attacker.

You can follow [1] to create xml redirect or use my template file [3]


##### References

[1] - https://nerdster.org/transparent-redirect-sdn/

[2] - https://docs.google.com/presentation/d/13K8Z1kl5XFZrWqBToMwFISSAPOKfzd3m9BtVcb-YAWs/edit#slide=id.g172a1d5889_16_16

[3] - https://gist.github.com/greatbn/24d846f3e0b9755d5e74e05cafd39ebd