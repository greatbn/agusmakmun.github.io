---
layout: post
title: Install Kubernetes using Kubeadm with calico network
image: "/content/images/2017/10/kubernetes.jpg"
date: '2017-10-10 04:09:00'
tags:
- k8s
- kubernetes
- docker
- kubeadm
---

These are my notes about install kubernetes using kubeadm with calico network plugin. 

**It's note, not a guide**

- Install Docker

```s
apt install docker.io -y
```

- Install Kubeadm


```s
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

- Init on Master node


```s
kubeadm init --pod-network-cidr=192.168.0.0/16
```

- Apply network plugins

```s
kubectl apply -f http://docs.projectcalico.org/v2.4/getting-started/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml
```

- Wait for dns ready


```s
root@sapd-kube-1:~# kubectl get pod -n kube-system
NAME                                       READY     STATUS              RESTARTS   AGE
calico-etcd-jw045                          0/1       ContainerCreating   0          55s
calico-node-640v5                          0/2       ContainerCreating   0          54s
calico-policy-controller-336633499-jhtqc   0/1       Pending             0          54s
etcd-sapd-kube-1                           1/1       Running             0          40s
kube-apiserver-sapd-kube-1                 1/1       Running             0          22s
kube-controller-manager-sapd-kube-1        1/1       Running             0          41s
kube-dns-2425271678-hd0mh                  0/3       Pending             0          1m
kube-proxy-x32x9                           1/1       Running             0          1m
kube-scheduler-sapd-kube-1                 1/1       Running             0          26s
```


- When ready, you can join a node, on my lab, It take 10 minutes to done.

```s
root@sapd-kube-1:~# kubectl get pod -n kube-system
NAME                                       READY     STATUS    RESTARTS   AGE
calico-etcd-jw045                          1/1       Running   0          9m
calico-node-640v5                          2/2       Running   0          9m
calico-policy-controller-336633499-jhtqc   1/1       Running   0          9m
etcd-sapd-kube-1                           1/1       Running   0          9m
kube-apiserver-sapd-kube-1                 1/1       Running   0          9m
kube-controller-manager-sapd-kube-1        1/1       Running   0          9m
kube-dns-2425271678-hd0mh                  3/3       Running   0          10m
kube-proxy-x32x9                           1/1       Running   0          10m
kube-scheduler-sapd-kube-1                 1/1       Running   0          9m
```

- Default kubernetes not schedule workload on master node

- On other node, You can join now,

```s
root@sapd-kube-2:~# kubeadm join --token 6d0bb9.efe2d2c83a6d7947 10.5.8.73:6443
[kubeadm] WARNING: kubeadm is in beta, please do not use it for production clusters.
[preflight] Running pre-flight checks
[preflight] Starting the kubelet service
[discovery] Trying to connect to API Server "10.5.8.73:6443"
[discovery] Created cluster-info discovery client, requesting info from "https://10.5.8.73:6443"
[discovery] Cluster info signature and contents are valid, will use API Server "https://10.5.8.73:6443"
[discovery] Successfully established connection with API Server "10.5.8.73:6443"
[bootstrap] Detected server version: v1.7.7
[bootstrap] The server supports the Certificates API (certificates.k8s.io/v1beta1)
[csr] Created API client to obtain unique certificate for this node, generating keys and certificate signing request
[csr] Received signed certificate from the API server, generating KubeConfig...
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/kubelet.conf"

Node join complete:
* Certificate signing request sent to master and response
  received.
* Kubelet informed of new secure connection details.

Run 'kubectl get nodes' on the master to see this machine join.
```


- Wait for node ready

```s
root@sapd-kube-1:~# kubectl get node
NAME          STATUS     AGE       VERSION
sapd-kube-1   Ready      17m       v1.7.5
sapd-kube-2   NotReady   1m        v1.7.5
sapd-kube-3   NotReady   1m        v1.7.5
```

