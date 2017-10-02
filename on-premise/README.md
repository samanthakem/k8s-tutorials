## Spinning Up Your First Kubernetes On-premise Cluster

First of all, what is a k8s on-premise cluster? It is a cluster which runs on computers on the premise (in the building), that means it runs on virtual machine, private cloud or baremetal. So a k8s cluster would not be operated by a cloud provider (ie. Azure, Google Cloud, AWS).

**Note:** we are going to use an OpenStack environment but it is not, repeating, it is not controlled by a OpenStack Cloud Provider that means k8s will know it is a VM but it will know that it is a **OpenStack** VM.

### Cluster Nodes

For this tutorial, we will have a cluster composed by 1 master and 1 work node, both should be launched on OpenStack with the following setup:

|Name| Image | IP Adress  | vCPUs | Disk |  RAM |
| ----- | -----| ----- | -----| ------| --------|
| master-node| ubuntu-16.04 | 10.11.4.37 | 2 | 40 | 2GB
| worker-node| ubuntu-16.04 | 10.11.4.136 | 1 | 20 | 1GB

### Setting up the Master node

In order to have a working k8s master node, you have to install a few packages:

1. Connect to the OpenStack VM instance ```master-node``` over SSH by running:

    ssh -i openstack_key.pem ubuntu@10.11.4.37

2. As root (```sudo su```), install the all the following packages:
    
    2.1 Install ```apt-transport-http``` to enable the usage of deb lines at ```/etc/source.list```:

        root@master-node:/home/ubuntu# apt-get update && apt-get install -y apt-transport-https

    2.2 Download the key used by ```apt``` to authenticate the package:

        root@master-node:/home/ubuntu# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

    2.3 Add the official k8s repository in ```/etc/apt/source.list```:

        $ cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
        > deb http://apt.kubernetes.io/ kubernetes-xenial main
        > EOF

    2.4 Update the index:

        root@master-node:/home/ubuntu# apt-get update

    2.5 Finally, install the k8s related packages:

        root@master-node:/home/ubuntu# apt-get install docker.io kubelet kubeadm kubectl kubernetes-cni

### Initializing Master node

Since you now have ```kubeadm``` installed in your node, it will be easy to initialize it.

Run:

    ubuntu@master-node:~$ sudo kubeadm init

Once you are done with this command, you will be asked to follow these instructions:

    ubuntu@master-node:~$ mkdir -p $HOME/.kube
    ubuntu@master-node:~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    ubuntu@master-node:~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

You then need to enable the networking and network policy in kubernetes across the cluster. Run:

```$ kubectl apply -f http://docs.projectcalico.org/v2.3/getting-started/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml```

Finally, check if everything is work fine by running:

```$ kubectl get pods --all-namespaces```

**Note:** If you want to watch the command so you don't have to run the same command several times in order to see if the resource is running/ready, just add ```watch``` before your command.

### Joining a node to the cluster

In order to add worker nodes to your cluster, you must follow the steps ```2.1``` to ```2.5``` for installing the packages and then ```join``` the newly created node to the cluster. To join your node to the cluster, run the following command in the worker node:

```kubeadmin join --token [CLUSTER-TOKEN] [MASTER-NODE-IP:PORT]```

If you don't remember the cluter token, run the following command into the master node:

```sudo kubeadm token list```

### Checking if the node were joined successfully:

To see if the join happened with no problem. run:

```kubectl get nodes```

You will see the status for every single node. Now you are able to manage your cluster using kubectl. For using kubectl from your local computer, you just need a copy of the ```$HOME/.kube/config``` file in your computer and make sure that master is accessible from your computer.
