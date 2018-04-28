
# [WORK IN PROGRESS]
This example is **not** final yet!
# 1. Create a cluster
To get started you will need to setup a Kubernetes cluster. In this example we will use Azure Container Service (ACS).

Follow [these steps](https://pascalnaber.wordpress.com/2017/09/12/run-net-core-2-docker-images-in-kubernetes-using-azure-container-service-and-azure-container-registry/) from [Pascal Naber's blog](https://pascalnaber.wordpress.com/) to setup a cluster. Note that you do not have to create a Azure Container Registry (ACR) because this example will use images that are publicly available from [Docker Hub ](https://hub.docker.com/). You can create an ACR if you want to use your own images.

# 2. Setup ingress routing
First we will have to deploy an ingress controller. There are different ingress controllers available but we will use the [NGINX ingress controller](https://github.com/kubernetes/ingress-nginx) which uses NGINX as a reverse proxy to route traffic to the right services. 

Run the command below to deploy 2 replicas of the readily available NGINX ingress controller image in our cluster.
```
kubectl apply -f .\ingress-controller\ingress-controller.deployment.yml
```

Then the command below to deploy the service required for our ingress controller. This service is of the type LoadBalancer and will have a public IP.
```kubectl apply -f .\ingress-controller\ingress-controller.service.yml```

# 3. Deploy "myapp"
Next we will have to deploy our app to our cluster. We will use the [Kubernetes Up And Running Demo (KUARD)](https://github.com/kubernetes-up-and-running/kuard) image for this. This is a readily available image that will show information about our cluster, about the node on the pod is running and about the pod itself. 

Run the command below to deploy 3 replicas of "myapp" into our cluster.
```kubectl apply -f .\ingress-controller\myapp.deployment.yml```

Next run the command below to deploy the "myappservice". This service is of the type NodePort and will not have a public IP.
```
kubectl apply -f .\ingress-controller\myapp.service.yml
```

To make the "myappservice" reachable from outside our cluster we will deploy an Ingress resource for it. This Ingress will tell our Ingress controller how to route traffic from to one of the 3 replicas. Run the command below to deploy it. 
```
kubectl apply -f .\ingress-controller\myapp.ingress.yml
```

The Ingress controller will automatically detect a change in the Ingress resources of our cluster and generate the required NGINX proxy configuration to route traffic to one of our myapp pods when someone connects to our cluster using ```http://<PUBLIC-IP>/myapp```