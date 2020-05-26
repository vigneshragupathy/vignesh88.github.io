---
layout: post
title: Kubernetes on Ubuntu 18.04 - Master and Dashboard setup
featured: true
date: '2018-06-14 12:11:00'
tags:
- linux
- opensource
- kubernetes
---

This post i am going to show how to install Kubernetes, configure Master node and enable Kubernetes dashboard in Ubuntu 18.04 LTS. I also tried to show the &nbsp;video demo explaining the entire configuration in the end of this post, This is my first video demo!!!

### Setup

I am using the Virtualbox(running in Ubuntu 18.04 physical machine) for this entire setup . The physical machine is Dell inspiron laptop with 12GB RAM , Intel® Core™ i7-6500U CPU @ 2.50GHz × 4 and 512GB SSD hardisk.

<!--kg-card-begin: image--><figure class="kg-card kg-image-card"><img src="/content/images/2018/06/setup.jpg" class="kg-image" alt="setup"></figure><!--kg-card-end: image-->
### Configuration

First install docker, it is provided by the default ubuntu 18.04 repository.

<!--kg-card-begin: code-->

    vikki@drona-child-1:~$ sudo apt-get install docker.io

<!--kg-card-end: code-->

Enable docker to auto start during reboot

<!--kg-card-begin: code-->

    vikki@drona-child-1:~$ sudo systemctl enable docker
    Synchronizing state of docker.service with SysV service script with /lib/systemd/systemd-sysv-install.
    Executing: /lib/systemd/systemd-sysv-install enable docker
    vikki@drona-child-1:~$

<!--kg-card-end: code--><!--kg-card-begin: code-->

    vikki@drona-child-1:~$ sudo apt-get install curl

<!--kg-card-end: code-->

Download the gpg key for kubernetes installation and add to ubuntu

<!--kg-card-begin: code-->

    vikki@drona-child-1:~$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
    OK

<!--kg-card-end: code-->

Now add the Google's kubernetes repository

<!--kg-card-begin: code-->

    vikki@drona-child-1:~$ sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"

<!--kg-card-end: code-->

Install kubeadm

<!--kg-card-begin: code-->

    vikki@drona-child-1:~$ sudo apt-get install kubeadm

<!--kg-card-end: code-->

If you want to disable the swap permanantly, edit /etc/fstab and comment the swap filesystem line. Kubernetes will not start if swap is enabled.

<!--kg-card-begin: code-->

    vikki@drona-child-1:~$ sudo swapoff -a

<!--kg-card-end: code-->

My IP address of the VM is 192.168.1.5 which i am going to advertise &nbsp;as apiserver and 40.168.0.0/16 is the newtwork for pod communication

<!--kg-card-begin: code-->

    vikki@drona-child-1:~$ sudo kubeadm init --pod-network-cidr=40.168.0.0/16 --apiserver-advertise-address=192.168.1.5
    [init] Using Kubernetes version: v1.10.4
    [init] Using Authorization modes: [Node RBAC]
    [preflight] Running pre-flight checks.
    	[WARNING SystemVerification]: docker version is greater than the most recently validated version. Docker version: 17.12.1-ce. Max validated version: 17.03
    	[WARNING FileExisting-crictl]: crictl not found in system path
    Suggestion: go get github.com/kubernetes-incubator/cri-tools/cmd/crictl
    [preflight] Starting the kubelet service
    [certificates] Generated ca certificate and key.
    [certificates] Generated apiserver certificate and key.
    [certificates] apiserver serving cert is signed for DNS names [drona-child-1 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.1.5]
    [certificates] Generated apiserver-kubelet-client certificate and key.
    [certificates] Generated sa key and public key.
    [certificates] Generated front-proxy-ca certificate and key.
    [certificates] Generated front-proxy-client certificate and key.
    [certificates] Generated etcd/ca certificate and key.
    [certificates] Generated etcd/server certificate and key.
    [certificates] etcd/server serving cert is signed for DNS names [localhost] and IPs [127.0.0.1]
    [certificates] Generated etcd/peer certificate and key.
    [certificates] etcd/peer serving cert is signed for DNS names [drona-child-1] and IPs [192.168.1.5]
    [certificates] Generated etcd/healthcheck-client certificate and key.
    [certificates] Generated apiserver-etcd-client certificate and key.
    [certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
    [kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/admin.conf"
    [kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/kubelet.conf"
    [kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/controller-manager.conf"
    [kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/scheduler.conf"
    [controlplane] Wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
    [controlplane] Wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
    [controlplane] Wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
    [etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/manifests/etcd.yaml"
    [init] Waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests".
    [init] This might take a minute or longer if the control plane images have to be pulled.
    [apiclient] All control plane components are healthy after 21.503420 seconds
    [uploadconfig] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
    [markmaster] Will mark node drona-child-1 as master by adding a label and a taint
    [markmaster] Master drona-child-1 tainted and labelled with key/value: node-role.kubernetes.io/master=""
    [bootstraptoken] Using token: o9an7t.o4bs1up74xjwnol3
    [bootstraptoken] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
    [bootstraptoken] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
    [bootstraptoken] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
    [bootstraptoken] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
    [addons] Applied essential addon: kube-dns
    [addons] Applied essential addon: kube-proxy
    
    Your Kubernetes master has initialized successfully!
    
    To start using your cluster, you need to run the following as a regular user:
    
      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config
    
    You should now deploy a pod network to the cluster.
    Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
      https://kubernetes.io/docs/concepts/cluster-administration/addons/
    
    You can now join any number of machines by running the following on each node
    as root:
    
      kubeadm join 192.168.1.5:6443 --token o9an7t.o4bs1up74xjwnol3 --discovery-token-ca-cert-hash sha256:548c922cf4f845f3dc6d7da407516652879c8a5085c87e0322770e1475105591
    vikki@drona-child-1:~$ 

<!--kg-card-end: code-->

Take a note of the join command, this will be used to join the other nodes to this kubernetes cluster.

The token generated is valid only for 24hours. In case the 24 hours exceed we need to generate the new token using the command "sudo kubeadm token create" .The current token can be view from master using the below command.

<!--kg-card-begin: code-->

    vikki@drona-child-1:~$ sudo kubeadm token list
    TOKEN TTL EXPIRES USAGES DESCRIPTION EXTRA GROUPS
    o9an7t.o4bs1up74xjwnol3 1h 2018-06-15T14:39:02+05:30 authentication,signing The default bootstrap token generated by 'kubeadm init'. system:bootstrappers:kubeadm:default-node-token
    vikki@drona-child-1:~$ 

<!--kg-card-end: code-->

The CA cert hash is used for the node to join in secure manner. This also can be view from the below command. Combaining this token and hash we can join the nodes.

<!--kg-card-begin: code-->

    vikki@drona-child-1:~$ openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
    548c922cf4f845f3dc6d7da407516652879c8a5085c87e0322770e1475105591
    vikki@drona-child-1:~$ 

<!--kg-card-end: code-->

Create the necessary directories

<!--kg-card-begin: code-->

    vikki@drona-child-1:~$ mkdir -p $HOME/.kube
    vikki@drona-child-1:~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    vikki@drona-child-1:~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

<!--kg-card-end: code-->

Now deploy the network for pod communications , i am using the flannel networking

<!--kg-card-begin: code-->

    vikki@drona-child-1:~$ sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
    clusterrole.rbac.authorization.k8s.io "flannel" created
    clusterrolebinding.rbac.authorization.k8s.io "flannel" created
    serviceaccount "flannel" created
    configmap "kube-flannel-cfg" created
    daemonset.extensions "kube-flannel-ds" created
    

<!--kg-card-end: code-->

Now wait fo kube-flanner-X to change to "Running" status. At this point all the pods should be in running status.

<!--kg-card-begin: code-->

    vikki@drona-child-1:~$ sudo kubectl get pods --all-namespaces
    NAMESPACE NAME READY STATUS RESTARTS AGE
    kube-system etcd-drona-child-1 1/1 Running 0 2m
    kube-system kube-apiserver-drona-child-1 1/1 Running 0 1m
    kube-system kube-controller-manager-drona-child-1 1/1 Running 0 2m
    kube-system kube-dns-86f4d74b45-lzzvv 3/3 Running 0 2m
    kube-system kube-flannel-ds-z5fkj 1/1 Running 0 1m
    kube-system kube-proxy-k4qv7 1/1 Running 0 2m
    kube-system kube-scheduler-drona-child-1 1/1 Running 0 2m
    vikki@drona-child-1:~$ 
    

<!--kg-card-end: code-->

Once the flannel network is deployed , we can verify the flannel interface for the &nbsp;ip address assigned

<!--kg-card-begin: code-->

    vikki@drona-child-1:~$ ip a show flannel.1
    5: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default 
        link/ether de:ab:2a:c3:7a:25 brd ff:ff:ff:ff:ff:ff
        inet 40.168.0.0/32 scope global flannel.1
           valid_lft forever preferred_lft forever
        inet6 fe80::dcab:2aff:fec3:7a25/64 scope link 
           valid_lft forever preferred_lft forever
    vikki@drona-child-1:~$ ip a show cni0
    6: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
        link/ether 0a:58:28:a8:00:01 brd ff:ff:ff:ff:ff:ff
        inet 40.168.0.1/24 scope global cni0
           valid_lft forever preferred_lft forever
        inet6 fe80::44ee:1cff:fe91:35b2/64 scope link 
           valid_lft forever preferred_lft forever
    vikki@drona-child-1:~$ 
    

<!--kg-card-end: code-->

Now deploy the kubernetes dashoard. Kubernetes dashboard is used to manage the kubernetes cluster using the web GUI interface.

<!--kg-card-begin: code-->

    vikki@drona-child-1:~$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
    
    secret "kubernetes-dashboard-certs" created
    serviceaccount "kubernetes-dashboard" created
    role.rbac.authorization.k8s.io "kubernetes-dashboard-minimal" created
    rolebinding.rbac.authorization.k8s.io "kubernetes-dashboard-minimal" created
    deployment.apps "kubernetes-dashboard" created
    service "kubernetes-dashboard" created
    vikki@drona-child-1:~$

<!--kg-card-end: code-->

Wait for kubernetes-dashboard-xxxx pods to goes "Running" status

<!--kg-card-begin: code-->

    vikki@drona-child-1:~$ sudo kubectl get pods --all-namespaces
    NAMESPACE NAME READY STATUS RESTARTS AGE
    kube-system etcd-drona-child-1 1/1 Running 0 3m
    kube-system kube-apiserver-drona-child-1 1/1 Running 0 3m
    kube-system kube-controller-manager-drona-child-1 1/1 Running 0 3m
    kube-system kube-dns-86f4d74b45-lzzvv 3/3 Running 0 4m
    kube-system kube-flannel-ds-z5fkj 1/1 Running 0 2m
    kube-system kube-proxy-k4qv7 1/1 Running 0 4m
    kube-system kube-scheduler-drona-child-1 1/1 Running 0 3m
    kube-system kubernetes-dashboard-7d5dcdb6d9-d6lvt 1/1 Running 0 39s
    vikki@drona-child-1:~$ 

<!--kg-card-end: code-->

Once the kubernetes-dashboard-xxxxxxx container goes to "Running" state we can start teh kubectly proxy with the below command. If you want to start only in localhost then you can change the address option to localhost

<!--kg-card-begin: code-->

    vikki@drona-child-1:~$ sudo kubectl proxy --address=0.0.0.0
    Starting to serve on [::]:8001
    
    

<!--kg-card-end: code-->

Now open the below kubernetes dashboard URL in browser

[http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/](http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/).

<!--kg-card-begin: image--><figure class="kg-card kg-image-card"><img src="/content/images/2018/06/vikki_kubernetes_dashboard_2.jpg" class="kg-image" alt="vikki_kubernetes_dashboard_2"></figure><!--kg-card-end: image-->

To properly login to the kubernetes dashboard we need to creat a service account and assign the proper role.

<!--kg-card-begin: code-->

    vikki@drona-child-1:~$ kubectl create serviceaccount dashboard -n default
    serviceaccount "dashboard" created
    vikki@drona-child-1:~$ 

<!--kg-card-end: code--><!--kg-card-begin: code-->

    vikki@drona-child-1:~$ kubectl create clusterrolebinding dashboard-admin -n default --clusterrole=cluster-admin --serviceaccount=default:dashboard
    clusterrolebinding.rbac.authorization.k8s.io "dashboard-admin" created
    vikki@drona-child-1:~$

<!--kg-card-end: code-->

Now the password to login kubernetes dashboard can be viewed by running the below command. Copy the decoded password and login to dashboard.

<!--kg-card-begin: code-->

    vikki@drona-child-1:~$ kubectl get secret $(kubectl get serviceaccount dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
    eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRhc2hib2FyZC10b2tlbi1nOGxnZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJkYXNoYm9hcmQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI1MDU1ZmE1ZC02ZmI0LTExZTgtOTgzZi0wODAwMjcxNjVmMDYiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkYXNoYm9hcmQifQ.LAdmbfTmx8HACuZy3hRJzS4rP7LH2PaSi85oeNnuP4tWbtSVqonnkXNrENsrODA0zE8Dh77rCrKWM3j3HElJF8Wy3TzYVa2aopBxbIrqZgwESfeuUhhjc5twEE_zrxyVpbch1-lU_ye29li2iYn1649yDYosxBhKsic9A3dOtGWJuiTmFTYrt1cwzKRC201yAIdxxP6BiIm_h8nK5pAw_TPpF1tOxo_P4Zy9msn8tDR1NM_GX5Q10FJO8Ef7zNgFr9jgh_b_BnCj61niiIKzGJYzXs0b3Q3nvohoGfeGavTpG1eMc26-rJ1MwCT1beOvdkJv4DyjJFENPcZaalY0ywvikki@drona-child-1:~$ 
    

<!--kg-card-end: code-->
### Video hands-on demo
<!--kg-card-begin: embed--><figure class="kg-card kg-embed-card"><iframe width="480" height="270" src="https://www.youtube.com/embed/79oLiDhZAm4?feature=oembed" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></figure><!--kg-card-end: embed-->

My next post on kubernetes [Kubernetes scaling the cluster](/kubernetes-growing-the-cluster-with-centos-7-node/)

