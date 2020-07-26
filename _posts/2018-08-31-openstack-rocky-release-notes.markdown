---
layout: post
title: Openstack Rocky release notes
date: '2018-08-31 02:19:36'
---

##  Rocky release notes


#### Cinder 

- Cinder now allows for capacity based QoS which can be useful in environments where storage performance scales with consumption (such as RBD backed storage). The newly added QoS specs are read_iops_sec_per_gb, write_iops_sec_per_gb, total_iops_sec_per_gb, read_bytes_sec_per_gb, write_bytes_sec_per_gb and total_bytes_sec_per_gb. These values will be multiplied by the size of the volume and passed to the consumer. For example, setting total_iops_sec_per_gb to 30 and -setting total_bytes_sec_per_gb to 1048576 (1MB) then creating a 100 GB volume with that QoS will result in a volume with 3,000 total IOPs and 100MB/s throughput limit.


- Dell EMC ScaleIO has been renamed to Dell EMC VxFlex OS. Documentation for the driver can be found under the new name. The driver maintains full backwards compatability with prior ScaleIO releases and no configuration changes are needed upon upgrade to the new version of the driver.



#### Neutron


- In order to reduce the time spent processing security group updates in the L2 agent, conntrack deletion is now performed in a set of worker threads instead of the main agent thread, so it can return to processing other events quickly.

- Support for floating IPs port forwarding has been added: Users can now forward the traffic from a TCP/UDP/other protocol port of a floating IP address to a TCP/UDP/other protocol port associated to one of the fixed IP addresses of a Neutron port.

- 



#### Octavia 

- Added ability for Octavia to automatically set Barbican ACLs on behalf of the user. Such enables users to create TLS-terminated listeners without having to add the Octavia keystone user id to the ACL list. Octavia will also automatically revoke access to secrets whenever load balancing resources no longer require access to them.

- Added UDP protocol support to listeners and pools.

- Members have a new boolean option backup. When set to true, the member will not receive traffic until all non-backup members are offline. Once all non-backup members are offline, traffic will begin balancing between the backup members.



#### Horizon

- Security groups now can be specified when creating a port. When the port security is enabled, the security groups tab will be displayed in create port workflow.

-  “Get me a network” feature provided by nova and neutron is now exposed in the launch server form. This feature will sets up a neutron network topology for a project if there is no network in the project. It simplifies the workflow when launching a server. In the horizon support, when there is no network which can be used for a server, a dummy network named ‘auto_allocated_network’ is shown in the network choices. The feature is disabled by default because it requires preparations in your neutron deployment. To enable it, set enable_auto_allocated_network in OPENSTACK_NEUTRON_NETWORK to True

- “Interfaces” tab is added to the instance detail page. The new tab shows a list of ports attached to an instance. Users now have an easy way to access the list of ports of the instance and edit security groups per port. In addition, “Edit Port Security Groups” menu is added as an action of the instance table.

