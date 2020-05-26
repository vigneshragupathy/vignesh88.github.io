---
layout: post
title: RBAC in kubernetes
date: '2019-11-28 16:26:19'
tags:
- kubernetes
- linux
- opensource
---

There are 3 elements involved in RBAC. In this post we are going to see how to provide user level access to resources.

- Subjects - Users or Process that wants access to Kubernetes API
- Resources - Kubernetes API objects like pods, deployments etc
- Verbs - Set of operations like get, watch create etc

### **Setup**

I am using the Virtualbox(running in Ubuntu 18.04 physical machine) for this entire setup . The physical machine is Dell inspiron laptop with 12GB RAM , Intel® Core™ i7-6500U CPU @ 2.50GHz × 4 and 512GB SSD hardisk.

<!--kg-card-begin: image--><figure class="kg-card kg-image-card"><img src="/content/images/2019/11/setup-6.jpg" class="kg-image"></figure><!--kg-card-end: image-->
##### Step 1: Create a private key

Create a new directly and navigate to the directory

<!--kg-card-begin: code-->

    vikki@kubernetes1:~$ mkdir ssl
    vikki@kubernetes1:~$ cd ssl/

<!--kg-card-end: code-->

Use openssl command a generate a private key _user1.key_

<!--kg-card-begin: code-->

    vikki@kubernetes1:~/ssl$ openssl genrsa -out user1.key 2048
    Generating RSA private key, 2048 bit long modulus
    .......................................................+++
    .......................................................................................................................................+++
    e is 65537 (0x10001)
    vikki@kubernetes1:~/ssl$ ls
    user1.key

<!--kg-card-end: code-->
##### Step 2: Generate a CSR 

Use the private key generated in the privous step and generate the certificate signing request(csr)

<!--kg-card-begin: code-->

    vikki@kubernetes1:~/ssl$ openssl req -new -key user1.key -out user1.csr -subj "/CN=user1/O=vikki.in"
    
    vikki@kubernetes1:~/ssl$ ls
    user1.csr user1.key

<!--kg-card-end: code-->
##### Step 3: Sign the CSR and generate certificate

The kubernetes cluster have the CA(certificate authority) key and certificate available under /etc/kubernetes/pki location

<!--kg-card-begin: code-->

    vikki@kubernetes1:~/ssl$ ls /etc/kubernetes/pki/ca.
    ca.crt ca.key  

<!--kg-card-end: code-->

Use the CA certificate and key to sign the CSR

<!--kg-card-begin: code-->

    vikki@kubernetes1:~/ssl$ sudo openssl x509 -req -in user1.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out user1.crt -days 365
    [sudo] password for vikki: 
    Signature ok
    subject=/CN=user1/O=vikki.in
    Getting CA Private Key
    
    vikki@kubernetes1:~/ssl$ ls
    user1.crt user1.csr user1.key

<!--kg-card-end: code-->

> Now we have the private key _user1.key_ and signed certificate _user1.crt_

##### Step 4: Set credential for the user

Now set credential for the user _user1_ with the private key and the signed certificate.

<!--kg-card-begin: code-->

    vikki@kubernetes1:~/ssl$ kubectl config set-credentials user1 --client-certificate=user1.crt --client-key=user1.key 
    User "user1" set.

<!--kg-card-end: code-->
##### Step 5: Set context for the user

Now set the new context with the username , cluster etc.

We can also map the context to specific namespace using the --namespacec option. By default it will take the default namespace

<!--kg-card-begin: code-->

    vikki@kubernetes1:~/ssl$ kubectl config set-context user1-context --cluster=kubernetes --user=user1
    Context "user1-context" created.
    
    vikki@kubernetes1:~/ssl$ kubectl config get-contexts 
    CURRENT NAME CLUSTER AUTHINFO NAMESPACE
    * kubernetes-admin@kubernetes kubernetes kubernetes-admin   
              user1-context kubernetes user1              

<!--kg-card-end: code-->

> Now we can see there are 2 context created.

##### Step 6: Create a role

Create a role and map the resources and verb required.

<!--kg-card-begin: code-->

    vikki@kubernetes1:~/ssl$ kubectl create role myrole --verb=get,create,list --resource=pods
    role.rbac.authorization.k8s.io/myrole created

<!--kg-card-end: code-->
##### Step 7: Create a rolebinding

Create a rolebinding and map the role and user.

<!--kg-card-begin: code-->

    vikki@kubernetes1:~/ssl$ kubectl create rolebinding myrolebinding --role=myrole --user=user1 
    rolebinding.rbac.authorization.k8s.io/myrolebinding created

<!--kg-card-end: code-->
##### Step 8: Verify role and rolebinding

List the role and rolebinding and verify both are created.

<!--kg-card-begin: code-->

    vikki@kubernetes1:~/ssl$ kubectl get role
    NAME AGE
    myrole 58s
    vikki@kubernetes1:~/ssl$ kubectl get rolebindings.rbac.authorization.k8s.io 
    NAME AGE
    myrolebinding 8s

<!--kg-card-end: code-->
##### Step 9: Change the context and verify role based access
<!--kg-card-begin: code-->

    vikki@kubernetes1:~/ssl$ kubectl config get-contexts 
    CURRENT NAME CLUSTER AUTHINFO NAMESPACE
    * kubernetes-admin@kubernetes kubernetes kubernetes-admin   
              user1-context kubernetes user1     

<!--kg-card-end: code-->

Switch to newly created contesxt _user1-context_

<!--kg-card-begin: code-->

    vikki@kubernetes1:~/ssl$ kubectl config use-context user1-context 
    Switched to context "user1-context".
    
    vikki@kubernetes1:~/ssl$ kubectl config get-contexts 
    CURRENT NAME CLUSTER AUTHINFO NAMESPACE
              kubernetes-admin@kubernetes kubernetes kubernetes-admin   
    * user1-context kubernetes user1           

<!--kg-card-end: code-->

> Now we can see the current context is switched to _user1-context._

Try to create a deployement

<!--kg-card-begin: code-->

    vikki@kubernetes1:~/ssl$ kubectl run nginx-deployment --image=nginx
    kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
    Error from server (Forbidden): deployments.apps is forbidden: User "user1" cannot create resource "deployments" in API group "apps" in the namespace "default"

<!--kg-card-end: code-->

> The deployment creation is failed because the new context has resource mapped only for pod.

Now lets create a pod and verify the status

<!--kg-card-begin: code-->

    vikki@kubernetes1:~/ssl$ vim pod.yaml

<!--kg-card-end: code--><!--kg-card-begin: html--><script src="https://gist.github.com/vignesh88/aa1503f5161a453f120ab6b121f6325a.js"></script><!--kg-card-end: html--><!--kg-card-begin: code-->

    vikki@kubernetes1:~/ssl$ kubectl create -f pod.yaml 
    pod/myapp-pod created

<!--kg-card-end: code--><!--kg-card-begin: code-->

    vikki@kubernetes1:~/ssl$ kubectl get pods
    NAME READY STATUS RESTARTS AGE
    httpd-7765f5994-vc2j5 1/1 Running 1 2d
    myapp-pod 0/1 ContainerCreating 0 5s
    nginx-7bffc778db-p4ff5 1/1 Running 1 2d

<!--kg-card-end: code-->

> We can see now the pod created successfully and running in the new context.

We can also test the permission of user using the below command

<!--kg-card-begin: code-->

    vikki@kubernetes1:~$ kubectl auth can-i create deployments --as user1
    no
    
    vikki@kubernetes1:~$ kubectl auth can-i create pods --as user1
    yes

<!--kg-card-end: code-->