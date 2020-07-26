---
layout: post
title: Install Openstack for Laborator using Ansible
image: "/content/images/2016/07/Screenshot-from-2016-07-31-21-44-46.png"
date: '2016-07-31 14:45:23'
tags:
- openstack
- ansible
- deploy-openstack-using-ansible
- install-openstack-using-ansible
- automation
---

#####Install Openstack for LabInstall Openstack for Laborator using Ansibleorator using Ansible

Recently I am studying with ansible and I am working with Openstack. When a new version of Openstack was released, I often install it manual. So I decided to learn ansible to deploy it at automation. 

I have writen a `ansible-playbook` to perform that. I have minimized complex of the playbook. 

The scenario is self service network with openvswitch.  I deploy that with one controller node and more compute nodes. 

You can [click here](http://docs.openstack.org/mitaka/networking-guide/scenario-classic-ovs.html) to know the traffic flow of package

<img src="http://docs.openstack.org/mitaka/networking-guide/_images/scenario-classic-general.png">

*The architecture..Source: Openstack Network Guide*

####How to play?

You have to install all requirements.

```
Vagrant 
Ansible (2.0.0.1 version)
Git
```

*Note: The ansible 2.1.0.0 have an error with openvswitch modules*

Let's clone my repo at github and switch branch

```
https://github.com/greatbn/openstack-ansible-install.git
cd openstack-ansible-install
git checkout virtualbox/mitaka
```

Use your favorite editor to edit `Vagrantfile` and `group_vars/all`

- Edit your bridge card

<img src="http://i.imgur.com/XrL6k1X.png">

You have to change bridge card associate with bridge card abovename in your computer. In here I use `wlp3s0` which you have to change.


- Edit IP and gateway for controller node

Open file `group_vars/all` and change `external_address_controller` is your controller IP (corresponding with bridge card above). At rest is `subnet mask` and `gateway`

```
external_address_controller: 192.168.21.111
external_netmask_controller: 255.255.255.0
external_gateway_controller: 192.168.21.1
```


Let's save these files. Open terminal, go to the repo and use vagrant to create your infrastructure.

```
vagrant up
```

After some minutes you can check status your infrastructure

<img src="http://i.imgur.com/LAo904q.png">

Then copy your public key to these servers. If you don't have it, you can generate by `ssh-keygen` command.

Use user/password: `vagrant`

```
ssh-copy-id vagrant@10.30.0.21
ssh-copy-id vagrant@10.30.0.22
```

Finally, Use ansible-playbook to deploy

```
ansible-playbook -i hosts -s site.yml
```

Have a coffee during install process.

<img src="http://i.imgur.com/HSVrzIb.png)">

Wait for install process finish and go to dashboard to create an network and boot an instance

Use user/password: `admin/saphi`
Horizon Address is external address which you setted above. 

My address is: `192.168.21.111`

<img src="http://i.imgur.com/SjHz4n4.png)">

After login

<img src="http://i.imgur.com/7ZMbjjZ.png)">

#####Summary

if you want to scale more compute nodes, You can edit `Vagrantfile` and `hosts` file. And rerun playbook.