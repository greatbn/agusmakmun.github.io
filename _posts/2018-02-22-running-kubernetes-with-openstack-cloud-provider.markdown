---
layout: post
title: Running kubernetes with Openstack Cloud Provider
image: "/content/images/2018/02/Untitled-drawing.jpg"
date: '2018-02-22 13:58:58'
tags:
- kubernets
- openstack
- kubeadm
---

## Kubernetes use Openstack as Cloud Provider

- In this guide, I will setup kubernetes (1.9.3) cluster by using kubeadm and use Openstack as a Cloud Provider 

### Pros 
- Use Cinder volume for persistent volume
- Use Octavia or LBaas Agent for Loadbalancer service type

### Cons

- Currently, we have to setup cloud credential manual and change args for kubelet in all host


#### Install Docker, Kubeadm , kubelet

The simplest way to install docker is use docker.io package from Ubuntu repository,

```
sudo apt install docker.io
```

I will use docker 1.13 


```
root@sapd-kube-master:~/charts/stable/grafana# docker version
Client:
 Version:      1.13.1
 API version:  1.26
 Go version:   go1.6.2
 Git commit:   092cba3
 Built:        Thu Nov  2 20:40:23 2017
 OS/Arch:      linux/amd64

Server:
 Version:      1.13.1
 API version:  1.26 (minimum version 1.12)
 Go version:   go1.6.2
 Git commit:   092cba3
 Built:        Thu Nov  2 20:40:23 2017
 OS/Arch:      linux/amd64
 Experimental: false
```

- Install kubeadm, kubelet latest, type following command

```
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat << EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

- After install these tool, we can check other requisites by run this command

```
kubeadm init --skip-preflight-checks
```

#### Create cloud-config file and change args for kubelet

- Change your credentials and run this command

```
cat << EOF > /etc/kubernetes/cloud-config
[Global]
auth-url=https://<your keystone api>:5000/v3
domain-name=Default
tenant-name=<your tenant name>
username=<your username>
password=<your password>
region=<your region>
EOF
```

- Note: currently we don't set any config about Loadbalancer, because loadbalancer has not deployed yet in cloud v2 (soon)

- Change args for kubelet

```
sed -i -E 's/(.*)KUBELET_KUBECONFIG_ARGS=(.*)$/\1KUBELET_KUBECONFIG_ARGS=--cloud-provider=openstack --cloud-config=\/etc\/kubernetes\/cloud-config \2/' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

- Restart kubelet


```
systemctl daemon-reload
systemctl restart kubelet
```

#### Create Kubernetes cluster

- Generate kubeadm config file


```
kind: MasterConfiguration
apiVersion: kubeadm.k8s.io/v1alpha1
cloudProvider: openstack
kubernetesVersion: 1.9.3
unifiedControlPlaneImage: "gcr.io/google_containers/hyperkube-amd64:v1.9.3"
networking:
  podSubnet: 192.168.0.0/16
```

Notes: change these options if you know
    - `cloudProvider: openstack`: use this config for Openstack Cloud
    - `podSubnet: 192.168.0.0/16`: use this for calico network

- Init cluster


```
kubeadm init --config /path/to/kubeadm-config.yml
```

- After `kube-apiserver` and `kube-controller-manager` running you can setup network plugin, in here we are using calico plugin, run following command

```
kubectl apply -f https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/kubeadm/1.7/calico.yaml
```

- After `kube-dns` is running, you can join worker node to master node

**Note: You have to copy cloud-config to other nodes and change kubelet args like master node**


#### Launch application use cinder volume

- Firstly, we have to create storageclass 

```
cat << EOF > cinder-storage-class.yml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: hdd1
provisioner: kubernetes.io/cinder
parameters:
  type: HDD1 # change for your cloud volume type
  availability: nova
EOF
```

- Create storage class

```
kubectl create -f cinder-storage-class.yml
```

- Use helm to install grafana, change storageClass in `values.yml` file

<img src="https://i.imgur.com/VbTOlWc.png">

 After install 


```
root@sapd-kube-master:~/charts/stable/grafana# helm ls
NAME    REVISION        UPDATED                         STATUS          CHART           NAMESPACE
grafana 1               Thu Feb 22 20:18:07 2018        DEPLOYED        grafana-0.7.0   default  
```

- Show pvc, pv

```
root@sapd-kube-master:~/charts/stable/grafana# kubectl get pvc,pv
NAME                  STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc/grafana-grafana   Bound     pvc-d25639fa-17d2-11e8-bfef-fa163e679d2b   1Gi        RWO            hdd1           36m

NAME                                          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                     STORAGECLASS   REASON    AGE
pv/pvc-d25639fa-17d2-11e8-bfef-fa163e679d2b   1Gi        RWO            Delete           Bound     default/grafana-grafana   hdd1                     36m
```



##### Reference

- https://sapham.net/install-kubernetes-using-kubeadm-with-calico-network-2/
- https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/
- https://kubernetes.io/docs/setup/independent/install-kubeadm/
- https://kubernetes.io/docs/concepts/storage/storage-classes/#openstack-cinder
- https://kubernetes.io/docs/concepts/cluster-administration/cloud-providers/#openstack
