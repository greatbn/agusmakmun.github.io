---
layout: post
title: Opendaylight 101 - Integrate with Openstack
date: '2019-05-18 12:42:37'
---

### Integrate Opendaylight with Openstack 

Recently, I have to write some sdn application, So I choose Opendaylight as a SDN Controller for my Cloud Environment.

It takes 4 days to integrate and VM can go to the internet. I have tried with Devstack but It did not enable L3 connectivity. 

In this post, I will integrate Opendaylight with Openstack. Please read carefully.

### Requirements / Preparation


- OpenStack Rocky or Stein
- OpenvSwitch 2.9.2
- Opendaylight Oxygen 0.8.4



### Network Model

<img src="https://i.imgur.com/37sMxdv.png">

We will use 3 interfaces for Controller and 2 interfaces for Compute Node. Because I will use Controller as Gateway node for external connectivity.

Details IP Address

- Controller:
    - ens3: 192.168.100.111/24 (access network)
    - ens9: None    (external Network)
    - ens10: 10.4.0.111/24 (management network)

- Compute
    - ens3: 192.168.100.112/24 (access network)
    - ens10: 10.4.0.112/24 (management network)

If you want to use multiple gateway nodes, You have to attach external interface for those nodes.


### Install Openstack

Install Openstack is out of the scope of this post, So you have to prepare a vanilla openstack.

Below is a list of service need to install on Controller and Compute

- Controller
    + keystone
    + glance
    + neutron-server, neutron-dhcp-agent, neutron-metadata-agent
    + nova-* except nova-compute
    + horizon (optional)
    + python-networking-odl
- Compute
    + nova-compute
    + python-networking-odl

### Install Opendaylight


Please refer my previous post to install Opendaylight, note use Oxygen release and OpenJDK 8


### Enable Feature for Opendaylight

I will set features on boot (as devstack). Open file `etc/org.apache.karaf.features.cfg` inside Opendayligt extracted folder.

Add `odl-neutron-service,odl-restconf-all,odl-aaa-authn,odl-dlux-core,odl-mdsal-apidocs,odl-dluxapps-nodes,odl-dluxapps-topology,odl-dluxapps-yangui,odl-dluxapps-yangvisualizer,odl-l2switch-switch,odl-l2switch-switch-ui,odl-ovsdb-hwvtepsouthbound-ui,odl-ovsdb-southbound-impl-ui,odl-netvirt-ui,odl-openflowplugin-flow-services-ui,odl-netvirt-openstack,odl-neutron-logger` to the end of line begin with `featuresBoot`


<img src="https://i.imgur.com/v9JISgh.png">


Start ODL


`./bin/karaf`



### Configure Neutron-server 

Open `/etc/neutron/neutron.conf` file, remove `router` in `service_plugins` option if you set. And add `odl-router_v2`

Use configure below for `/etc/neutron/plugins/ml2/ml2_conf.ini` 


```
[DEFAULT]
[ml2]
tenant_network_types = vxlan
extension_drivers = port_security
mechanism_drivers = opendaylight_v2

[ml2_type_flat]
flat_networks = external

[ml2_type_vxlan]
vni_ranges = 1:1000

[securitygroup]
firewall_driver = neutron.agent.not.a.real.FirewallDriver

[ml2_odl]
enable_dhcp_service = False
port_binding_controller = pseudo-agentdb-binding
password = admin
username = admin
url = http://10.4.0.111:8181/controller/nb/v2/neutron
```

Then restart neutron-server 



### Configure OpenVswitch on Controller

So now, You have to configure some parameters for openvswitch

- Hostconfig: in recent version of opendaylight/networking-odl we need parameters to know more about a node in SDN network such as which node will host router,
which node has DPDK and so on.

- Controller: in the concept of SDN network, Sdn switch will connect to Sdn controller 


*Note: The latest version of networking-odl ship with command `neutron-odl-ovs-hostconfig` but in Rocky release you have to use script `set_ovs_hostconfigs.py` in `/usr/lib/python2.7/dist-packages/networking_odl`*

Because this node will host router so we have to option `ODL L3`

Run below command


```
python set_ovs_hostconfigs.py --noovs_dpdk  --ovs_hostconfigs='{"ODL L2":{"allowed_network_types":["local","vlan", "vxlan","gre"],"bridge_mappings": {"external":"br-ex"},"supported_vnic_types": [{"vnic_type":"normal","vif_type":"ovs","vif_details":{}}]},"ODL L3": {}}'
```

*Note: the external network will map with `br-ex` bridge*


Add other configs


```
python set_ovs_hostconfigs.py --noovs_dpdk
```

Add bridge `br-ex` and set `local_ip` for tunnel


```
ovs-vsctl add-br br-ex
ovs-vsctl set Open_vSwitch . other_config:local_ip=10.4.0.111
ovs-vsctl set Open_vSwitch . other_config:provider_mappings=external:br-ex
```


Now connect switch with OpenDaylight


```
ovs-vsctl set-manager ptcp:6641:127.0.0.1 tcp:10.4.0.111:6640
```

*Note: We will open port 6641 for openvswitch to neutron-dhcp-agent can talk with openvswitch. If you run Opendaylight in other host you do not have to open port 6641 for openvswitch.*


Check all options for table Open_vswitch

<img src="https://i.imgur.com/yBfvaNZ.png">

### Configure OpenVswitch on Compute Node

Configure hostconfig


```
python set_ovs_hostconfigs.py --noovs_dpdk  --ovs_hostconfigs='{"ODL L2":{"allowed_network_types":["local","vlan", "vxlan","gre"],"bridge_mappings": {},"supported_vnic_types": [{"vnic_type":"normal","vif_type":"ovs","vif_details":{}}]}}'
```

Add other parameters



```
python set_ovs_hostconfigs.py --noovs_dpdk
```

Set `local_ip` for tunnel

```
ovs-vsctl set Open_vSwitch . other_config:local_ip=10.4.0.112
```

Connect Switch to Opendaylight


```
ovs-vsctl set-manager ptcp:6641:127.0.0.1 tcp:10.4.0.111:6640
```
### Configure Nova-compute 

Add these config to `nova.conf` file and restart nova-compute 

```
[os_vif_ovs]
ovsdb_connection = tcp:127.0.0.1:6641
```

### Configure Neutron-dhcp-agent 


Add paramter for `ovsdb_connection` and force metadata to run inside `qdhcp-xxx` namespace


```
[DEFAULT]
interface_driver = openvswitch
force_metadata = true


[ovs]
ovsdb_connection = tcp:127.0.0.1:6641

```


### Check Network Agent


<img src="https://i.imgur.com/Mjl3eEF.png">

### Summary


- You need to define ODL L3 for the gateway node

- Not sure about the latest version of Opendaylight and OpenvSwitch

- If you see any misconfigure please tell me