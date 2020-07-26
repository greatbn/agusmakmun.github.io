---
layout: post
title: OpendayLight 101 - Installation
date: '2018-10-02 03:58:38'
tags:
- opendaylight
- sdn
- software-defined-network
---

##### Install OpendayLight

- Install java on host


```
sudo apt install default-jdk -y
```

- Download Zip file, I will use Boron release


```
wget https://nexus.opendaylight.org/content/repositories/opendaylight.release/org/opendaylight/integration/distribution-karaf/0.5.1-Boron-SR1/distribution-karaf-0.5.1-Boron-SR1.tar.gz
```

- Extract 


```
tar -xvf distribution-karaf-0.5.1-Boron-SR1.tar.gz
```

- Run Opendaylight


```
cd distribution-karaf-0.5.1-Boron-SR1
./bin/karaf
```

It takes some minutes to start please wait

<img src="https://i.imgur.com/q2ONYSi.png">

Now We will install Dlux UI to visualize and OpenFlow plugin for later post

Type `feature:list | grep dlux` to check plugin is available


<img src="https://i.imgur.com/B5IVIYR.png">


##### Install Dlux

```
feature:install odl-dlux-all
```

Once you run the previous comamnd, Dlux will be enabled and you find that Opendaylight server has started listening on additions ports:

+ 8181: Web UI and MD-SAL RESTCONFIG

+ 8185: AAA
+ 8080: Web UI and MD-SAL RESTCONFIG

You can access through your browser by entering `http://your_odl_id:8181/index.html`


You can login to Dlux using the following default credentials

```
Username: admin
Password: admin
```


#####  Install Openflow Plugin 

- Install L2 Switch module


```
feature:install odl-l2switch-switch
```

Once installed, Opendaylight server started listening on port 6633 for OpenFlow Protocol