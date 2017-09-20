# Spinning Up Your First Kubernetes Cluster on Azure

For this tutorial, we are going to need [Azure CLI](https://github.com/Azure/azure-cli/) which is an Azure command-line tool. 

## Installation

Type the following command in you terminal:

        curl -L https://aka.ms/InstallAzureCli | bash

You are going to be asked to answer some configurations options such as installation directory, etc.

Once you are done with the installation, run ```az -v``` and you will see info about the current version of Azure CLI in your machine.

**Note:** It you have problems with the ```az``` command and it fails with a meaningless error, add  a  debug flag: ```--debug```

## Login into your Azure Account

In order to use all the azure commands, you will need to be logged in into an Azure account. If you don't have an Azure account, [create one](https://azure.microsoft.com/en-in/free/).

Once you create your account, type the following command:

        az login

This command will ask you to follow this instruction:

```To sign in, use a web browser to open the page https://aka.ms/devicelogin and enter the code [YOUR-CODE] to authenticate.```

Open the given link in your browser, put the code given on the right field and insert your azure account info. If everything goes right, you will now be able to manipulate your Azure infrastructure.

Check you credentials to be sure:

        $ az account list

        [
          {
            "cloudName": "AzureCloud",
            "id": [YOUR-ID],
            "isDefault": true,
            "name": "Free Trial",
            "state": "Enabled",
            "tenantId": [YOUR-TENANT-ID],
            "user": {
              "name": [YOUR-EMAIL],
              "type": "user"
            }
          }
        ]

Now you are done with setting up.

## Next step: create an actual k8s cluster on Azure

This tutorial will show you how to create a cluster in ```west europe``` named ```k8s-cluster-eu``` with only 1 agent.

### First, create a Resource Group...

Resource groups provide a way to monitor, control access, provision and manage billing for collections of assets that are required to run an application, or used by a client or company department. According to the [official documentation](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview), resource group is a container that holds related resources for an Azure solution. To create a resource group, run:

        az group create --name az-west-eu --location westeurope
    
where:

* ```az-west-eu``` is the name of the resource.
<br/>
* ```westeurope``` is the resource location.

Once you finish running this command, you will get:

        {
          "id": "/subscriptions/[RESOURCE-ID]/resourceGroups/az-west-europe",
          "location": "westeurope",
          "managedBy": null,
          "name": "az-west-europe",
          "properties": {
            "provisioningState": "Succeeded"
          },
          "tags": null
        }

That means you successfully created a resource group on Azure.

**Note:** You can see other locations by running:

```az account list-locations```

### Finally, creating a k8s cluster using the created resource group

Run:

    az acs create --orchestrator-type kubernetes --resource-group az-west-eu --name k8s-cluster-eu --generate-ssh-keys --agent-count 1

where:

* ```--resource-group``` defines which resource group will be refered.
* ```--name``` defines the cluster name.
* ```--generate-ssh-keys``` generates (dâ) ssh keys in your machine for authentication purposes

## Authorizing k8s client to operate with your cluster

In order for you to operate the cluster created on Azure, you need to get the credentials to connect the ```kubectl``` with the cluster.

To do so, run:

    az acs kubernetes get-credentials --resource-group [YOUR-RESOURCE-GROUP-NAME] --name [YOUR-CLUSTER-NAME]

When it finishs, you will note that the kubernetes config file located in ```~/.kube/config``` will contain the credentials to operate on the cluster.

Now, let's make sure everything is working fine and that you have access to the cluster through the k8s client.

Run the following command to see if the clusters were added to your kubernetes configuration:

    kubectl config get-contexts

It should be able to see the cluster that you just created.

Now, if you want so see the resources allocated to the cluster, run:

    kubectl get pods --all-namespaces

So... that's it, you now have a up and running cluster on you Azure provider and it is possible to manage it by using Kubernetes client (kubectl). Yay! :smile:

## Optional: Delete the Kubernetes Cluster

In case you want to delete your cluster, run:

    az acs delete --name [YOUR-CLUSTER-NAME] --resource-group [RESOURCE-GROUP-CONTAINING-YOUR-CLUSTER]

## Optional: Delete the Resource Group

Run: 

    az group delete --name [RESOURCE-GROUP-CONTAINING-THE-CLUSTER]

**Note:** When you delete the resource group, you also delete resources other than a cluster, becareful with this command...








