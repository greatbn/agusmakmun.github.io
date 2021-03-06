---
layout: post
title: 'Openstack manila phần 2: Generic Driver'
image: "/content/images/2016/06/openstack.jpg"
date: '2017-01-02 13:49:00'
tags:
- openstack
- manila
- shared-file-system
- storage
---

#####Cấu hình manila share với Generic Driver

Yêu cầu khi sử dụng Generic Driver là bạn phải có cấu hình nova, neutron và cinder trong file cấu hình `/etc/manila/manila.conf` trên node manila-share

Và trên node manila share sẽ cần cài đặt L2 Agent. ở đây mình sử dụng OpenvSwitch. Mình sẽ cài đặt manila-share trên node compute để không phải cấu hình lại openvswitch.

Mô hình cài đặt

<img src="http://i.imgur.com/D0Zk5uP.png">

- Ta sẽ cần một image có sẵn share service. Tải image về từ openstack foundation

```
wget http://tarballs.openstack.org/manila-image-elements/images/manila-service-image-master.qcow2

```

- Upload image và tạo flavor.

```
openstack image create "manila-service-image" \
--file manila-service-image-master.qcow2 \
--disk-format qcow2 \
--container-format bare \
--public

openstack flavor create manila-service-flavor --id 100 --ram 256 --disk 0 --vcpus 1
```

- Kiểm tra image

```
openstack image list
```

<img src="http://i.imgur.com/xQSLoyY.png">

- Thêm các mục sau vào `/etc/manila/manila.conf`. *Sửa lại password các user*

```
[nova]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = saphi

[cinder]

auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = cinder
password = saphi

[neutron]

url = http://controller:9696
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = saphi
```

- Khai báo backend sử dụng, Thêm vào phần [DEFAULT] như sau

```
enabled_share_backends= generic1 #generic1 là tên phần backend
enabled_share_protocols=NFS,CIFS #giao thức cho phép
```

- Khai báo cấu hình backend

```
[generic1]
share_backend_name = GENERIC-1 
share_driver = manila.share.drivers.generic.GenericShareDriver 
driver_handles_share_servers = True 
service_instance_flavor_id = 100
service_image_name = manila-service-image
service_instance_user = manila 
service_instance_password = manila
#interface_driver = manila.network.linux.interface.BridgeInterfaceDriver # nếu sử dụng LinuxBridge thì uncomment
interface_driver = manila.network.linux.interface.OVSInterfaceDriver
```
- Reset lại dịch vụ manila-share

```
service manila-share restart
```

- Tương tự ta kiểm tra trạng thái các service manila

```
manila service-list
```

<img src="http://i.imgur.com/qf1e3sm.png">

- Tạo share-type và share-network

```
manila type-create generic True
```


True ở đây có nghĩa là `Driver Handling Share Servers = True`

<img src="http://i.imgur.com/MOt0QWR.png">

- Với Share-network ta sẽ cần net-id và subnet-id của project.

```
root@controller:/home/saphi# neutron net-list
+--------------------------------------+------------------------+------------------------------------------------------+
| id                                   | name                   | subnets                                              |
+--------------------------------------+------------------------+------------------------------------------------------+
| 85caa7b4-1212-4037-b350-5e2db386b416 | private_demo           | f5c151cd-5baa-4628-b8dc-43963115adc7 192.168.2.0/24  |
| 56fc4f98-ddc9-49a0-a014-5b86f7d99f82 | ext-net                | a1bb678f-d630-4c1a-8053-7af2676e2b94 172.16.25.0/24  |
| 9af94508-21a7-4c3f-b126-04adf719ec19 | private-net            | a6b97594-0338-4f99-beea-cd0d0d07363b 192.168.10.0/24 |
+--------------------------------------+------------------------+------------------------------------------------------ 
```
Mình đang ở project admin do vậy sẽ lấy subnet và net id của admin là dải `private-net`

```
root@controller:/home/saphi# manila share-network-create --name share-net-admin --neutron-subnet-id a6b97594-0338-4f99-beea-cd0d0d07363b --neutron-net-id 9af94508-21a7-4c3f-b126-04adf719ec19
+-------------------+--------------------------------------+
| Property          | Value                                |
+-------------------+--------------------------------------+
| name              | share-net-admin                      |
| segmentation_id   | None                                 |
| created_at        | 2016-06-05T11:02:17.209356           |
| neutron_subnet_id | a6b97594-0338-4f99-beea-cd0d0d07363b |
| updated_at        | None                                 |
| network_type      | None                                 |
| neutron_net_id    | 9af94508-21a7-4c3f-b126-04adf719ec19 |
| ip_version        | None                                 |
| nova_net_id       | None                                 |
| cidr              | None                                 |
| project_id        | 2dbc33e520b84b8ab550b649099d7972     |
| id                | 9a295209-63d7-4637-9efc-f6317e7bff8a |
| description       | None                                 |
+-------------------+--------------------------------------+
```

Kiểm tra share network đã tạo ra

<img src="http://i.imgur.com/XhbbA8Y.png">

Ta cũng thấy rằng sẽ có một mạng và subnet mới được tạo ra  với tên `manila_service_network `

<img src="http://i.imgur.com/j9cZc7i.png">


Thực hiện tạo share server sử dụng giao thức NFS và kích thước share 1GB có tên share-01

```
manila create nfs 1 --share-network share-net-admin --name share-01 
```

Và sau đó ta sẽ có share server

<img src="http://i.imgur.com/Ldol4lM.png">

Mình đang có một instance `sa` trên project admin có IP `192.168.10.7`
```
root@controller:/home/saphi# openstack server list
+--------------------------------------+------+--------+--------------------------+
| ID                                   | Name | Status | Networks                 |
+--------------------------------------+------+--------+--------------------------+
| dbf12e7f-1fec-4817-8a39-e7100835f092 | sa   | ACTIVE | private-net=192.168.10.7 |
+--------------------------------------+------+--------+--------------------------+
```
Tạo access-allow cho instance trên vào share `share-01`

<img src="http://i.imgur.com/NNSLOy8.png">

Để lấy đường dẫn mount `manila show share-01`

```
root@controller:/home/saphi# manila show share-01
+-----------------------------+-----------------------------------------------------------------------+
| Property                    | Value                                                                 |
+-----------------------------+-----------------------------------------------------------------------+
| status                      | available                                                             |
| share_type_name             | default_share_type                                                    |
| description                 | None                                                                  |
| availability_zone           | nova                                                                  |
| share_network_id            | 9a295209-63d7-4637-9efc-f6317e7bff8a                                  |
| export_locations            |                                                                       |
|                             | path = 10.254.0.12:/shares/share-d7287607-4750-4874-b3ca-bd9c734cee0e |
|                             | preferred = False                                                     |
|                             | is_admin_only = False                                                 |
|                             | id = 0db97dd9-0af2-49fb-907e-854263727995                             |
|                             | share_instance_id = d7287607-4750-4874-b3ca-bd9c734cee0e              |
| share_server_id             | 25c6e835-5dbe-4bbb-8451-29df0805aaef                                  |
| host                        | compute1@generic#GENERIC                                              |
| access_rules_status         | active                                                                |
| snapshot_id                 | None                                                                  |
| is_public                   | False                                                                 |
| task_state                  | None                                                                  |
| snapshot_support            | True                                                                  |
| id                          | fd6574cc-ac43-4f8d-bfd1-22edbf1bf113                                  |
| size                        | 1                                                                     |
| name                        | share-01                                                              |
| share_type                  | 4d7aca95-a7d0-4d66-9036-7e7e876d8c4c                                  |
| has_replicas                | False                                                                 |
| replication_type            | None                                                                  |
| created_at                  | 2016-06-05T11:06:50.000000                                            |
| share_proto                 | NFS                                                                   |
| consistency_group_id        | None                                                                  |
| source_cgsnapshot_member_id | None                                                                  |
| project_id                  | 2dbc33e520b84b8ab550b649099d7972                                      |
| metadata                    | {}                                                                    |
+-----------------------------+-----------------------------------------------------------------------+
```
Trên instance `sa` thưc hiện mount

<img src="http://i.imgur.com/mqCH11c.png">


Các phần tiếp theo:

[Phần 2: Generic Driver](https://sapham.net/openstack-manila-phan-2-generic-driver/)

[Phần 3: LVM Driver](https://sapham.net/openstack-manila-phan-3-lvm-driver/)

[Phần 4: GlusterFS Native Driver](https://sapham.net/openstack-manila-phan-4-native-glusterfs-driver/)