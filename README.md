# Set up a MultiMaster Setup for Kubernetes using Kubeadm utility
>**Nodes Requirements** 

Here I am setting up a basic cluster with 2 master nodes and 1 worker node and 1 load balancer node to load balance b/w to masters to create highly available cluster
* 1 LoadBalancer Node using HAProxy
* 2 Master Nodes
* 1 Worker Node

> **Setting up load balancer node.**

**Install HAProxy** 
```sudo yum update && sudo yum upgrade -y ```
![image](https://user-images.githubusercontent.com/71398798/144403767-59090c46-0bf3-4a0a-a8a3-d4147c3616c9.png)

```sudo yum install haproxy```<br/>
![image](https://user-images.githubusercontent.com/71398798/144404302-ff9796db-41c5-4a27-b952-3c84407ed89d.png)

**Now Edit HAProxy Config**
``` sudo vi /etc/haproxy/haproxy.cfg ```


We have to create frontend and backend service for our HAProxy, so from our frontend service we will use the ip of loadbalancer to use kubectl and loadbalancer will implicitly loadbalanced b/w the ips of our master nodes using the backend service we will provide.
Here we have to provide the ips of our master01 and master02 nodes and the default port for the kube proxy API server i.e 6443 so that loadbalancer can communicate with the master nodes using kubectl.

> **For Frontend** 

```
frontend frontend-apiserver
      bind 0.0.0.0:6443
      mode tcp
      option tcplog
      default_backend backend_apiserver
```


> **For Backend**

``` 
backend backend_apiserver
     mode tcp
     option tcp-check
     balance roundrobin
     default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
        server <master node hostname> <master node ip:6443> check
        server <master node 02 hostname> <master node 02 ip:6443> check
```

  
![image](https://user-images.githubusercontent.com/71398798/144407669-6ec68c80-d2ce-4c2d-97ba-e8fca5d334d7.png)

> **Now add entry for master01 and master02 in /etc/hosts file

``` sudo vi /etc/hosts ```

![image](https://user-images.githubusercontent.com/71398798/144408075-a151c310-db85-4a1d-9c5b-e70b45184961.png)


>**Now restart and verify HAProxy**

``` sudo systemctl restart haproxy```

``` sudo systemctl status haproxy```

If you are getting this error use this command
![image](https://user-images.githubusercontent.com/71398798/144410794-39fa4089-b326-4ce7-9d8b-fc82df64f24a.png)

```sudo setsebool -P haproxy_connect_any=1```

![image](https://user-images.githubusercontent.com/71398798/144410981-293c5935-5a2b-42a8-9fac-d6c7f743531c.png)

Install netcat

``` sudo yum install -y nc ```

Run to confirm **6443** is binding to **loadbalancer**

``` nc -v localhost 6443```

![image](https://user-images.githubusercontent.com/71398798/144411474-bb1962e4-a593-4922-b047-96612c710cfb.png)

Add rule to firewalld otherwise cluster won't be able to communicate with the loadbalancer node

```sudo firewall-cmd --zone=public --permanent --add-port=<exposed lb port>/tcp```
```sudo systemctl restart firewalld```
```sudo systemctl daemon-reload```

>**Check If kube proxy api server is up**

**Type in any of the master node**
```curl https://<loadbalancer ip>:<exposed port>/version -k```
If there is no output or no route to host then debug firewall config
If successful you will get prompt similar to below

![image](https://user-images.githubusercontent.com/71398798/144432284-cdc984ab-b5fc-42c4-88eb-6009f62d3067.png)




>**PRE REQUISITES**
>*Kubeadm, kubelet, kubectl and docker must be installed on all the nodes i.e master01, master02, worker 
>*Swap must be off use ```sudo swapoff -a``` and edit in ```vi /etc/fstab``` for persistance
>*If you have init a cluster before run ``` sudo kubeadm reset -f ``` and flush iptables
>*I am using calico network system for different network list you are free to use any manifest provided by the kubernetes
>*Disable firewalld service ```sudo systemctl stop firewalld and sudo systemctl disable firewalld```


**BOOTSTRAPPING K8 Cluster**
On any of the master node initialize the cluster

```kubeadm init --control-plane-endpoint=<LOADBALANCER_IP:PORT> --upload-certs --pod-network-cidr=<Calico Network Range>```
![image](https://user-images.githubusercontent.com/71398798/144412933-068e755f-88eb-4b70-9ef7-30916e4bc273.png)



After successful execution you will get a prompt similar to this

![image](https://user-images.githubusercontent.com/71398798/144432936-6771423c-d027-4074-b01c-440023a20a4f.png)

Please keep note of the join token for master, worker nodes

**Copy the join token for master and paste it on master02**
>Master 02 has also join the cluster

![image](https://user-images.githubusercontent.com/71398798/144433738-0c8ed94b-15b9-44d2-944b-c5e0c9709629.png)


Execute the provided **three** commands on both the masters

![image](https://user-images.githubusercontent.com/71398798/144434325-1ad6daaf-5d0b-461c-a356-2710174be04f.png)

**ON LOADBALANCER NODE**
>Follow this procedure

*Create Kube directory ```mkdir -p $HOME/.kube```
*If not root add user permission for the file so that it can be copied using scp ```sudo chown centos /etc/kubernetes/admin.conf```
*Use ip of master01 and use ssh to copy admin.conf to loadbalancer node ```scp centos@master01:/etc/kubernetes/admin.conf $HOME/.kube/config ```

![image](https://user-images.githubusercontent.com/71398798/144435778-0eb8495b-39d0-4ab9-9589-4b71f8361d55.png)


*Finally give owner ship to copied file ```sudo chown $(id -u):$(id -g) $HOME/.kube/confi```

>Now load balancer is almost setup you need to install kubectl cli so that we can communicate with the master nodes here i have it already setup

Here multimaster setup is done I can see multiple control-plane-nodes

![image](https://user-images.githubusercontent.com/71398798/144436125-9a67bcac-a48f-4cef-a085-a3c61a84c305.png)


**IMPORTANT**
**It says not ready because i've not setup any network cni for the internal pod traffic, you can setup any network from kubernetes and then they will be in ready state**



