## Setting Up Nginx Ingress Controller in On-premise Cluster

Hi, if you're reading this tutorial, you're most likely interested to get to know how to setup a nginx ingress controller in on-premise clusters, so let's dive into it.

In this tutorial, I am assuming that you have a kubernetes on-premise cluster. If you don't have one, follow this [tutorial](https://github.com/samanthakem/k8s-tutorials/tree/master/on-premise) to setup an on-premise cluster in OpenStack environment.

### Installing Helm (k8s package manager)

For simplicity, I used [helm](https://github.com/kubernetes/helm) to deploy the ingress controller. Helm is a Kubernetes package manager that allows to install charts (packages of pre-configured Kubernetes resources).

In your local machine, install **helm**:

    brew install kubernetes-helm

Now, do ssh into your cluster master node and run:

    helm init --context on-prem-cluster

By running this command you will be able to comunicate with your cluster from your local machine. For more information, access the [official documentation](https://docs.helm.sh/using_helm/#installing-helm).

To see all the resources related to ```tiller```, run:

    kubectl get all --namespace=kube-system -l name=tiller

You will get as output, something like:

    NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    deploy/tiller-deploy   1         1         1            1           2m

    NAME                         DESIRED   CURRENT   READY     AGE
    rs/tiller-deploy-cffb976df   1         1         1         2m

    NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    deploy/tiller-deploy   1         1         1            1           2m

    NAME                         DESIRED   CURRENT   READY     AGE
    rs/tiller-deploy-cffb976df   1         1         1         2m

    NAME                               READY     STATUS    RESTARTS   AGE
    po/tiller-deploy-cffb976df-v44nc   1/1       Running   0          2m

    NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
    svc/tiller-deploy   ClusterIP   10.106.252.210   <none>        44134/TCP   2m

### Installing Nginx Ingress Controller chart

Now that you have **helm** installed in your machine and **tiller** setup in your cluster, youb are able to install charts. To install the [nginx ingress controller chart](https://github.com/kubernetes/charts/tree/master/stable/nginx-ingress), run:

    helm install stable/nginx-ingress --name ingress -f values.yaml

Where ```values.yaml``` are defined in [its official repo](https://github.com/kubernetes/charts/blob/master/stable/nginx-ingress/values.yaml) or [here](). If you are getting the ```values.yaml``` from the official repository, make sure you change the service type to ```NodePort``` and define the nodePorts since on-premise clusters do not support ```LoadBalancer``` services. To do so, uncomment [this](https://github.com/kubernetes/charts/blob/master/stable/nginx-ingress/values.yaml#L110) and delete (or comment out) [this](https://github.com/kubernetes/charts/blob/master/stable/nginx-ingress/values.yaml#L108).

If everything goes right, you will get an output similar to:

    NAME:   ingress
    LAST DEPLOYED: Fri Oct  6 01:03:04 2017
    NAMESPACE: default
    STATUS: DEPLOYED

    RESOURCES:
    ==> v1/ConfigMap
    NAME                              DATA  AGE
    ingress-nginx-ingress-controller  1     2s

    ==> v1/Service
    NAME                                   CLUSTER-IP      EXTERNAL-IP  PORT(S)                     AGE
    ingress-nginx-ingress-controller       10.103.140.205  <nodes>      80:32080/TCP,443:32443/TCP  2s
    ingress-nginx-ingress-default-backend  10.101.39.0     <none>       80/TCP                      2s

    ==> v1beta1/Deployment
    NAME                                   DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
    ingress-nginx-ingress-controller       1        1        1           0          2s
    ingress-nginx-ingress-default-backend  1        1        1           0          1s


    NOTES:
    The nginx-ingress controller has been installed.
    Get the application URL by running these commands:
    export HTTP_NODE_PORT=32080
    export HTTPS_NODE_PORT=32443
    export NODE_IP=$(kubectl --namespace default get nodes -o jsonpath="{.items[0].status.addresses[1].address}")

    echo "Visit http://$NODE_IP:$HTTP_NODE_PORT to access your application via HTTP."
    echo "Visit https://$NODE_IP:$HTTPS_NODE_PORT to access your application via HTTPS."

    An example Ingress that makes use of the controller:

    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
        annotations:
        kubernetes.io/ingress.class: nginx
        name: example
        namespace: foo
    spec:
        rules:
        - host: www.example.com
            http:
            paths:
                - backend:
                    serviceName: exampleService
                    servicePort: 80
                path: /
        # This section is only required if TLS is to be enabled for the Ingress
        tls:
            - hosts:
                - www.example.com
            secretName: example-tls

    If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

    apiVersion: v1
    kind: Secret
    metadata:
        name: example-tls
        namespace: foo
    data:
        tls.crt: <base64 encoded cert>
        tls.key: <base64 encoded key>
    type: kubernetes.io/tls

### Creating Ingress resources

Yay! We have a Nginx Ingress Controller up and running in our cluster, now we are able to create Ingress Resources but first, let's create some services...

Follow [this]() tutorial to acomplish this task so we can move forward to the main part which is handling Ingress resources. When you finish this tutorial, you will have three working services:

    frontend-service
    backend-service
    microbot

Now, for the next step which is actuallt creating the ```Ingress``` resource, we will need a DNS to use as hosts. Let's use [xip.io](http://xip.io/) for demo purposes.

Use something like:

    webapp.10.103.140.205.xip.io
    microbot.10.103.140.205.xip.io

Where the ip in the above dns belongs to the nginx ingress controller, in order to get it, run:

```kubectl get svc ingress-nginx-ingress-controller```

Get the cluster ip! :)

In our first ```Ingress``` resource, we will have [this]():

    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
    name: microbot
    spec:
    rules:
    - host: microbot.10.103.140.205.xip.io
        http:
        paths:
            - path: /
            backend:
                serviceName: microbot
                servicePort: 80

Run ```kubectl create -f microbot.yaml``` and you will create the ```Ingress``` resource.

Now, to verify if the ingress is really working, try running ```curl microbot.10.103.140.205.xip.io``` **within your cluster** to see what happens... hopefully, if everything workes fine, you will see this:

    <!DOCTYPE html>
    <html>
    <style type="text/css">
        .centered
        {
        text-align:center;
        margin-top:0px;
        margin-bottom:0px;
        padding:0px;
        }
    </style>
    <body>
        <p class="centered"><img src="microbot.png" alt="microbot"/></p>
        <p class="centered">Container hostname: microbot-548c4cd85f-87sbj</p>
    </body>
    </html>

And that means your ingress is working! :D

Now, let's create other ```Ingress``` resource:

    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
    name: webapp
    annotations:
        # This tells to only use the Nginx Ingress Controller
        # and avoids the creation on a Global LoadBalancer on GKE.
        kubernetes.io/ingress.class: "nginx"
    spec:
    rules:
    - host: webapp.10.103.140.205.xip.io
        http:
        paths:
        - path: /
            backend:
            serviceName: frontend-service
            servicePort: 80
        - path: /testback
            backend:
            serviceName: backend-service
            servicePort: 80

Run ```kubectl create -f webapp.yaml``` and you will create the ```Ingress``` resource.

Now, to verify if the ingress is really working, try running ```curl webapp.10.103.140.205.xip.io``` and ```curl webapp.10.103.140.205.xip.io/testback``` **within your cluster** to see what happens...

Hopefully you wil hit the service endpoints and will see the output as expected.

**Note:** If you try to hit any other path, you will get the 404 error.

So yes guys, now you have an on-prem cluster up and running with ingress resources working fine! :D I hope this tutorial was helpful... 


