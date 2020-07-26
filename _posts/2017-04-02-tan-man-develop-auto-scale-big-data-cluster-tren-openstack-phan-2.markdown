---
layout: post
title: "[Tản mạn]  Develop Auto Scale Big Data Cluster trên Openstack phần 2"
image: "/content/images/2017/05/OpenStack-Hadoop-Cloud.png"
date: '2017-04-02 17:39:40'
tags:
- openstack
- cloud
- big-data
- openstack-sahara
- auto-scale
- hadoop
---

Tiếp nối [phần 1](https://sapham.net/develop-auto-scale-big-data-cluster-tren-openstack/), mình đã nói về việc chỉnh sửa mã nguồn của `sahara-dashboard`. 
Quay trở lại với hàm `cluster_create` của `saharadashboard/api/sahara`. Hàm này sử dụng `python-saharaclient` để gửi request tạo cluster tới `sahara-api`, vì mình sẽ gửi thêm 3 args nữa là `is_autoscale`, `max_cpu` và `max_ram` do vậy cũng cần chỉnh sửa hàm `clusters.create` của `python-saharaclient`

Hàm này nằm tại tập tin `saharaclient/api/clusters` 

Thêm các tham số mới 

<img src="http://i.imgur.com/AMScWx9.png)">

Thêm các tham số khi client gửi lên `sahara-api`

<img src="http://i.imgur.com/4VU1TCx.png">

Nếu muốn sử dụng command line để tuơng tác ta có thể sửa tiếp tập tin `saharaclient/osc/v1/clusters.py`

Tham khảo commit mình đã sửa 

- https://github.com/greatbn/python-saharaclient/commit/abb64b76f9900464fbb0f6c274c723b5d8b84a25
- https://github.com/greatbn/python-saharaclient/commit/1885e99061d847c260ebc80794dcef814fdd0501

Vậy là client đã gửi đi các tham số cần thiết, vẫn theo workflow này, ta cũng cần chỉnh sửa `sahara-api` xử lý các thông tin này và lưu vào database. 
Ta sẽ cần tạo thêm 3 cột mới trong bảng `clusters`. 
Openstack giúp ta việc này rẩt đơn giản bằng cách sử dụng migration đatabase tool là `alembic`, ta chỉ việc khai báo các cột cần thêm vào bảng nào. 

Thêm file `033_add_autoscale.py` vào folder `sahara/db/migration/alembic_migrations/versions` với nội dung như sau:

<img src="http://i.imgur.com/29VjLa5.png)">

Lưu ý: cần khai báo `revision` (phiên bản mới ta sẽ sửa), `down_revision` (phiên bản mà ta sẽ sửa) đúng.

```
revision = '033'
down_revision = '032'
```

Tiếp tục thêm các trường mới vào `models.py` để sahara conductor hiểu được các columns mới trong table `clusters`

Mở tập tin `sahara/db/sqlalchemy/models.py`

Sửa class `Clusters` thêm các trường mới




```python
class Cluster(mb.SaharaBase):
    """Contains all info about cluster."""

    __tablename__ = 'clusters'

    __table_args__ = (
        sa.UniqueConstraint('name', 'tenant_id'),
    )

    id = _id_column()
    name = sa.Column(sa.String(80), nullable=False)
    description = sa.Column(sa.Text)
    tenant_id = sa.Column(sa.String(36))
    trust_id = sa.Column(sa.String(36))
    is_transient = sa.Column(sa.Boolean, default=False)
    plugin_name = sa.Column(sa.String(80), nullable=False)
    hadoop_version = sa.Column(sa.String(80), nullable=False)
    cluster_configs = sa.Column(st.JsonDictType())
    default_image_id = sa.Column(sa.String(36))
    neutron_management_network = sa.Column(sa.String(36))
    anti_affinity = sa.Column(st.JsonListType())
    management_private_key = sa.Column(sa.Text, nullable=False)
    management_public_key = sa.Column(sa.Text, nullable=False)
    user_keypair_id = sa.Column(sa.String(80))
    status = sa.Column(sa.String(80))
    status_description = sa.Column(st.LongText())
    info = sa.Column(st.JsonDictType())
    extra = sa.Column(st.JsonDictType())
    rollback_info = sa.Column(st.JsonDictType())
    sahara_info = sa.Column(st.JsonDictType())
    use_autoconfig = sa.Column(sa.Boolean(), default=True)
    provision_progress = relationship('ClusterProvisionStep',
                                      cascade="all,delete",
                                      backref='cluster',
                                      lazy='joined')
    verification = relationship('ClusterVerification', cascade="all,delete",
                                backref="cluster", lazy='joined')
    node_groups = relationship('NodeGroup', cascade="all,delete",
                               backref='cluster', lazy='joined')
    cluster_template_id = sa.Column(sa.String(36),
                                    sa.ForeignKey('cluster_templates.id'))
    cluster_template = relationship('ClusterTemplate',
                                    backref="clusters", lazy='joined')
    shares = sa.Column(st.JsonListType())
    is_public = sa.Column(sa.Boolean())
    is_autoscale = sa.Column(sa.Boolean())
    max_cpu = sa.Column(sa.Integer())
    max_ram = sa.Column(sa.Integer())

    is_protected = sa.Column(sa.Boolean())
    domain_name = sa.Column(sa.String(255))
```

Sahara Conductor sẽ xử lý các request tới sahara và ghi vào database, do vậy ta sẽ modify để conductor thêm các dữ liệu mới của chúng ta vào dữ liệu hiện tại mà conductor ghi vào database

Thêm vào hàm `cluster_create` của `sahara/conductor/manager.py` như sau

<img src="http://i.imgur.com/w2Dz8iY.png">

Vậy là conductor sẽ ghi vào database đúng như kì vọng. Nhưng vẫn chưa xong, mỗi khi có request POST gửi lên `sahara-api` tạo cluster, `sahara-api` sẽ gọi tới các hàm check dữ liệu này, nếu dữ liệu không theo form đó thì sẽ không thể tạo cluster

<img src="http://i.imgur.com/JOZphGQ.png)">

Tiếp tục chỉnh các template của dữ liệu gửi lên. Mở tập tin `sahara/service/validations/cluster_template_schema.py`

Sửa dict `CLUSTER_TEMPLATE_SCHEMA`

Thêm các `key` và `value` như sau

<img src="http://i.imgur.com/Ct9z5Ty.png)">

Về cơ bản là ta đã có thể chạy được code này

- thực hiện tạo virtualenv 
- upgrade head database
- run `sahara-api` và `sahara-engine`
- cài đặt mã nguồn `sahara-dashboard` mới
- `collectstatic` và `compress`

Vậy là mình đã xong 2 bài tản mạn, mô tả lại quá trình modify code sahara của mình, các bạn có thể tham khảo github của mình tại branch `stable/ocata`

- https://github.com/greatbn/sahara/
- https://github.com/greatbn/sahara-dashboard/
- https://github.com/greatbn/python-saharaclient/