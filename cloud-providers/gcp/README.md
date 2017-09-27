# Spinning up your first Kubernetes cluster on GKE

## Create a GCP Account

Make sure you have a Google Cloud Platform account before continuing this tutorial. You can use an existing GCP account or [sign up](https://console.cloud.google.com/freetrial) for a new one with a valid Gmail account.

**Note**: A credit card is required for Google Cloud Platform, but it is free to subscribe, you will get free credit to use with any resourcers that you need. However, once these credits are over, you will need to pay to continue using the google resources.

## Installing Google Cloud SDK

Although you are able to use [Google Cloud Shell](https://cloud.google.com/shell/docs/) which provides you an easy way for you to manage your Cloud Platform Console projects and resources straight from your browser without having to install the Google Cloud SDK and other tools on your system, in this tutorial, we are going to use [Google Cloud SDK](https://cloud.google.com/sdk/). With Cloud Shell, the Cloud SDK gcloud command and other utilities you need are always available when you need them.

The following steps will get you ready to use the Cloud SDK! You can also find out how to install it from [Google's official documentation](https://cloud.google.com/sdk/docs/quickstarts).

**1.** Download the latest release by typing:

        curl https://sdk.cloud.google.com | bash

**2.** You will be asked some questions (installation directory)

**3.** Restart your terminal

**4.** Make sure your SDK is successfully installed by typing the following command which will give you the path where the sdk was installed:

        which gcloud

Now you are able to use `gcloud` command. :raised_hands: :raised_hands:


## Create a project

A GCP project is required for this tutorial. You can use an existing GCP project or [create a new one](https://console.cloud.google.com/projectcreate?organizationId=0). Make sure you save your `PROJECT_ID` somewhere since this info is going to be very important in next steps.

**Note:** Your project name maybe different from your project id.

## Enable Compute Engine and Container Engine APIs

In order to create the cloud resources required by this tutorial, you will need to enable the following APIs using the Google API Console:

* [Compute Engine API](https://console.cloud.google.com/apis/api/compute_component/overview)

* [Container Engine API](https://console.cloud.google.com/apis/api/container/overview)

## Authenticating and setting up your Google Cloud environment

In order for you to authenticate, you have to run:

```gcloud auth login```

You will get an URL, go there and put your auth credentials.

## Preparing Deployment Environment

Now that we have ```gcloud``` command-line tool all set, we can now set the project that we created as default. Run:

```gcloud config set project [PROJECT-ID]```

**Note:** You can see your configurations by running ```gcloud config list```.

## Let's create a cluster

You can create a cluster on the Container Clusters section on the project dashboard by going to Container Engine > Container Clusters > New container cluster or if you want, just running:

```gcloud container clusters create [CLUSTER-NAME] --zone [ZONE] --num-nodes [NUM-NODES]```

Once it is done, you can set this cluster as your default:

```gcloud config set container/cluster [CLUSTER-NAME]```

## Getting cluster credentials

Now that we have a k8s cluster that will contain our application, we must notify our local setup. Run:

```gcloud container clusters get-credentials [CLUSTER-NAME] --zone [ZONE] --project [PROJECT-ID]```

## Checking the configurations

Finally, check all your configurations are all good by running:

```kubectl config get-contexts```

Also, verify if you are able to see all the resources:

```kubectl get pods --all-namespaces```

# Deploying an app

To create our Google Cloud platform Docker image, we have to follow some rules before it is pushed to the Google Container Registry. The image has to be created in the following format: ```gcr.io/[PROJECT-ID]/[IMAGE]:[TAG]```

Check this [link](https://github.com/samanthakem/hello-go) which is a simple Go application with Dockerfile on it.

In order to build the image, run:

```docker build -t gcr.io/[PROJECT-ID]/hello-go:0.0.1 .``` inside the cloned repo.

With our image ready, pushing it to the Container Registry is simply:

```gcloud docker -- push gcr.io/[PROJECT-ID]/hello-go:0.0.1```

This may take a while but once it's done, we should have the image listed on Compute > Container Engine > Container Registry. which can be managed/orchestrated by Kubernetes!


## Deployment

All we have now is a Docker image on the Container Registry. To actually run the application, we need to create an instance from the image - A Docker container. To achieve this, we will create a Kubernetes Deployment which creates a pod for our container.

There are two ways to create a deployment using the kubectl command. You can either specify the parameters in a yml file or the command-line.

**Command-line**

Run:


```kubectl run [DEPLOYMENT_NAME] --image=gcr.io/[PROJECT_ID]/[IMAGE]:[TAG] --port=[PORT]```

**File**

Check the ```deployment.yml``` file to see the code.

To create a deployment, run:

```kubectl create -f deployment.yml```

You can view the deployment and the created pods by running kubectl get deployments and kubectl get pods respectively. Adding a -w enables us to see how the pods are created.

## Service

By creating a service, you will expose an application to external traffic. Currently, the deployment is only accessible within the Kubernetes cluster.

```A Kubernetes Service is an abstraction which defines a logical set of Pods and a policy by which to access them - sometimes called a micro-service. It enables external traffic exposure, load balancing and service discovery for those Pods. ```

Just like a deployment, you can create a service from a them command-line or from a file.

**Command-line**

Run:

```kubectl expose deployment [SERVICE_NAME] --type="LoadBalancer"```

**File**

Check the ```service.yml``` file to see the code.

To create the service, run:

```kubectl create -f service.yml```

Once the service is created, we can get the externally accessible IP address by listing all the services. To do so, run: ```kubectl get services```. The external IP may take a few seconds to be visible. 