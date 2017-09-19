# Spinning Up Your First Kubernetes Cluster on Azure

For this tutorial, we are going to need [Azure CLI](https://github.com/Azure/azure-cli/) which is an Azure command-line tool. 

## Installation

Type the following command in you terminal:

        curl -L https://aka.ms/InstallAzureCli | bash

You are going to be asked to answer some configurations options such as installation directory, etc.

Once you are done with the installation, run ```az -v``` and you will see info about the current version of Azure CLI in your machine.

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

This tutorial will show you how to create a cluster in ```west europe``` named ```az-europe``` with only 1 agent.

### First, create a Resource Group...

Resource groups provide a way to monitor, control access, provision and manage billing for collections of assets that are required to run an application, or used by a client or company department. According to the [official documentation](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview), resource group is a container that holds related resources for an Azure solution. To create a resource group, run:

        az group create --name az-west-europe --location westeurope
    
where:

```az-west-europe``` is the name of the resource.
<br/>
```westeurope``` is the resource location.

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










