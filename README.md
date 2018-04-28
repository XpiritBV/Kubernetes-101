
# [WORK IN PROGRESS]
This example is **not** final yet!
# 1. Create a cluster
To get started you will need to setup a Kubernetes cluster. In this example we will use Azure Container Service (ACS).

Open a command prompt and run:
```
az login
```

Next run:
```
az account list
```

Make sure you select the right Azure subscription, you can change it by running:
```
az account set --subscription <subscription-id>
```

Once you have selected the right subscribtion run the command below to create a new resoucre group for our cluster:
```
az group create --name "introduction-to-k8s" --location "westeurope"
```

Next run the following commadn to create a k8s cluster in Azure Container Service (ACS):
```
az acs create --orchestrator-type kubernetes --name k8scluster --resource-group introduction-to-k8s --agent-count 3 --generate-ssh-keys
```

It will take a few minutes for this command to complete because it has to setup an entire cluster for you. Once it is done you will have a Kubernetes cluster named **k8scluster** (*--name k8scluster*) inside a resource group named **introduction-to-k8s** (*--resource-group introduction-to-k8s*) with 1 master and **3 nodes** (*--agent-count 3*). The **SSH keys required to connect to the cluster** have been generated automatically (*--generate-ssh-keys*).

Next we will download an install kubectl. First check if you don't have it already by running:
```
kubectl
```

If ```kubectl``` is not found, run:
```
az acs kubernetes install-cli
```

Next we need connect to the cluster by running:
```
az acs kubernetes get-credentials --name k8scluster --resource-group introduction-to-k8s
```

Lastly verify we have a connection by running:
```
kubectl get nodes
```

# 2. Setup ingress routing
First we will have to deploy an ingress controller. There are different ingress controllers available but we will use the [NGINX ingress controller](https://github.com/kubernetes/ingress-nginx) which uses NGINX as a reverse proxy to route traffic to the right services. 

Run the command below to deploy 2 replicas of the readily available NGINX ingress controller image in our cluster.
```
kubectl apply -f .\ingress-controller\ingress-controller.deployment.yml
```

Then the command below to deploy the service required for our ingress controller. This service is of the type LoadBalancer and will have a public IP.
```
kubectl apply -f .\ingress-controller\ingress-controller.service.yml
```

# 3. Deploy "myapp"
Next we will have to deploy our app to our cluster. We will use the [Kubernetes Up And Running Demo (KUARD)](https://github.com/kubernetes-up-and-running/kuard) image for this. This is a readily available image that will show information about our cluster, about the node on the pod is running and about the pod itself. 

Run the command below to deploy 3 replicas of "myapp" into our cluster.
```
kubectl apply -f .\ingress-controller\myapp.deployment.yml
```

Next run the command below to deploy the "myappservice". This service is of the type NodePort and will not have a public IP.
```
kubectl apply -f .\ingress-controller\myapp.service.yml
```

To make the "myappservice" reachable from outside our cluster we will deploy an Ingress resource for it. This Ingress will tell our Ingress controller how to route traffic from to one of the 3 replicas. Run the command below to deploy it. 
```
kubectl apply -f .\ingress-controller\myapp.ingress.yml
```

The Ingress controller will automatically detect a change in the Ingress resources of our cluster and generate the required NGINX proxy configuration to route traffic to one of our myapp pods when someone connects to our cluster using ```http://<PUBLIC-IP>/myapp```