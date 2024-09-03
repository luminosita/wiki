# **Quickly setup a K3s cluster in one minute using k3sup**


![alt text](_assets/1_b8JA75C-5F600U4Y7YkpjA.webp)

Photo by Pixabay from Pexels

### Background

In my daily work, in order to facilitate testing in a pure environment, I often need to frequently build and destroy clusters in local or public cloud environments. Sometimes in my HomeLab environment, although the CPU is not powerful, it is better because the memory is large enough. Later, after I got the Azure credit given by Microsoft MVP, I would often set up in Azure virtual machines because there was no network problem of pulling the image. .

In both environments, I used Terraform to quickly create and destroy virtual machines, and then created K3s clusters on the virtual machines. K3s cluster is lightweight enough and supports customization of components. Combined with  [Alfred Snippets](https://www.alfredapp.com/help/features/snippets/), I only need to ssh to the virtual machine and type  `k3si`  to quickly enter customized commands, and then obtain the  `kubeconfig`  file on the virtual machine and replace the api in it -server addresses (these are also resolved via snippet):

```bash
$ export MASTER_IP=${MASTER_IP:-$(ip addr show eth0 | grep 'inet ' | awk '{print $2}' | cut -d/ -f1)}  

$ export INSTALL_K3S_VERSION=v1.23.8+k3s1  

$ curl -sfL https://get.k3s.io | sh -s - --disable traefik --disable local-storage --disable metrics-server --advertise-address=$MASTER_IP --disable servicelb --write-kubeconfig-mode 644 --write-kubeconfig ~/.kube/config
```

A single-node cluster is quite convenient to operate, but when you need a multi-node cluster, you still need to ssh to all hosts for operation. Of course, you need to copy the token of the master node. It’s still a bit cumbersome.

Then I discovered a faster tool,  [k3sup](https://github.com/alexellis/k3sup)  (pronounced ‘ketchup’) created by Alex Ellis.

### Introduction to k3sup

k3sup is a lightweight tool used to quickly build K3s clusters.

k3sup features ease of use, requiring only a single command to install K3s on different platforms. It enables users to quickly create Kubernetes clusters and easily join new nodes to existing clusters.

k3sup connects to the target server via SSH and automatically installs and configures K3s. This means we can install and run Kubernetes on any machine that can be accessed via SSH, including local machines, cloud servers, or devices like Raspberry Pi.

A simple understanding is that k3sup is used to complete a series of operations such as ssh to the host, install K3s server, copy token, ssh to the agent host, install K3s agent… etc.

Next we look at how to use k3sup.

### Install k3sup

k3sup is a command line tool. You need to download and install the CLI before using it.

Linux:

```bash
$ curl -sLS https://get.k3sup.dev | sh  
$ sudo install k3sup /usr/local/bin/
```

macOS:

brew install k3sup

### Usage

k3sup supports the following commands:

-   `completion`：Generates an autocomplete script for the specified shell
-   `help`：Help
-   `install`：Install K3s on the server via SSH
-   `join`：Install the K3s agent on the remote host and join it to the existing cluster
-   `ready`：Use kubectl to check whether the cluster is ready.
-   `update`：Print update instructions
-   `version`：print version

Creating a cluster will use the  `install`  and  `join`  commands.

### Install

The  `install`  command is used to install K3s on the server. Use the following command to install k3s on the remote host.

Where  `--ip`  points to the address of the remote host,  `--user`  is the username to log in to the remote host,  `--k3s-channel`  is the version to be installed,  `--local-path`  The local storage address of the cluster kubeconf. More options can be viewed via  `k3sup help install`  .

> _k3sup uses ssh key_ `_~/.ssh/id_rsa_` _by default to access the host, which can be specified through the_ `_--ssh-key_` _option._

```bash
$ export MASTER_IP=192.168.1.11  
$ k3sup install --ip $MASTER_IP \  
    --user addo \  
    --k3s-channel v1.24  \  
    --local-path /tmp/config
```
Executing the command will print the logs during the installation process.

```
Running: k3sup install  
2023/10/26 09:04:35 192.168.1.11  
Public IP: 192.168.1.11  
[INFO]  Finding release for channel v1.24  
[INFO]  Using v1.24.17+k3s1 as release  
...  
Saving file to: /tmp/config
```

### Test your cluster with:  
```bash
$ export KUBECONFIG=/tmp/config  
$ kubectl config use-context default  
$ kubectl get node -o wide
```
Execute the command to view node information.

```bash
$ export KUBECONFIG=/tmp/config  
$ kubectl get node -o wide  
NAME     STATUS   ROLES                  AGE   VERSION         INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME  
master   Ready    control-plane,master   1m   v1.24.17+k3s1   10.0.2.4      <none>        Ubuntu 20.04.6 LTS   5.15.0-1047-azure   containerd://1.7.3-k3s1
```
If you are installing a single-node cluster, the  `install`  command is sufficient. If it is a multi-node cluster, you also need to use the  `join`  command.

### Join

Use the  `join`  command to initialize the agent node and add it to the current cluster. You need to use  `--server-ip`  to specify the IP address of the server node. You also need to specify  `--k3s-channel`  The installed version, it is strongly recommended to install the same version on the server node.

```bash
$ export AGENT_IP=192.168.1.12  
$ k3sup join --ip $AGENT_IP --user addo --server-ip $MASTER_IP --k3s-channel v1.24
Running: k3sup join  
Agent: 192.168.1.11 Server: 192.168.1.12  
Received node-token from 192.168.1.11.. ok.  
[INFO]  Finding release for channel v1.24  
[INFO]  Using v1.24.17+k3s1 as release  
...
```

Check nodes:

```bash
$ kubectl get no   
NAME     STATUS   ROLES                  AGE     VERSION  
node-1   Ready    <none>                 43s   v1.24.17+k3s1  
master   Ready    control-plane,master   2m58s   v1.24.17+k3s1
```
### Full Scripts

With the help of ChatGPT, generate a script to create a cluster with one click. You can try how long it takes to create a two-node cluster. I tried it and it took about 32 seconds.

### Define IP addresses  
```bash
$ export HOSTS="192.168.1.11 192.168.1.12"
```
### Setup Cluster

```bash 
#!/bin/bash  
  
# Read the list of IP addresses from the environment variable  
IP_ADDRESSES=($HOSTS)  
# Define the k3s version  
K3S_VERSION="v1.24"  
  
# Check if there is at least one IP address  
if [ ${#IP_ADDRESSES[@]} -eq 0 ]; then  
    echo "No IP addresses found. Please ensure the HOSTS environment variable is correctly set."  
    exit 1  
fi  
  
# Install the master node  
MASTER_IP=${IP_ADDRESSES[0]}  
echo "Installing master node: $MASTER_IP"  
k3sup install --ip $MASTER_IP --user addo --k3s-channel $K3S_VERSION \  
    --k3s-extra-args '--write-kubeconfig-mode 644 --write-kubeconfig ~/.kube/config --disable traefik --disable metrics-server --disable local-storage --disable servicelb' \  
    --local-path /tmp/config  
  
# Install the other agent nodes  
for i in "${!IP_ADDRESSES[@]}"; do  
    if [ $i -ne 0 ]; then  
        AGENT_IP=${IP_ADDRESSES[$i]}  
        echo "Installing agent node: $AGENT_IP"  
        k3sup join --ip $AGENT_IP --server-ip $MASTER_IP --user addo --k3s-channel $K3S_VERSION  
    fi  
done  
  
echo "k3s cluster installation complete."
```

### Cleanup Cluster

```bash
#!/bin/bash  
  
# Read the list of IP addresses from the environment variable  
IP_ADDRESSES=($HOSTS)  
# Check if there is at least one IP address  
if [ ${#IP_ADDRESSES[@]} -eq 0 ]; then  
    echo "No IP addresses found. Please ensure the HOSTS environment variable is correctly set."  
    exit 1  
fi  
# Clean up the master node  
MASTER_IP=${IP_ADDRESSES[0]}  
echo "Cleaning up master node: $MASTER_IP"  
ssh -i ~/.ssh/id_rsa $MASTER_IP k3s-uninstall.sh  
# Clean up the other agent nodes  
for i in "${!IP_ADDRESSES[@]}"; do  
    if [ $i -ne 0 ]; then  
        AGENT_IP=${IP_ADDRESSES[$i]}  
        echo "Cleaning up agent node: $AGENT_IP"  
        ssh -i ~/.ssh/id_rsa $AGENT_IP k3s-agent-uninstall.sh  
    fi  
done  
echo "k3s cluster cleanup complete."
```