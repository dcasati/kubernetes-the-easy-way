# ACS and ACS-Engine - What is it all about ?

## Azure Container Service

According to it's [definition](https://docs.microsoft.com/en-us/azure/container-service/): 

> Azure Container Service allows you to quickly deploy a production ready Kubernetes, DC/OS, or Docker Swarm cluster.

In other words, Azure Container Service (ACS) is essentially a turn-key solution that allow the deployment of containers with various orchestration engines. Take Kubernetes as an example. ACS takes away all of the deployment, upgrade and management tasks that are part of spinning up a Kubernetes cluster such as certificates and PKI creation and distribution to the controllers and worker nodes.

> ACS is a turn-key solution to quickly deploy containers on Azure.

So then, what is Azure Container Service Engine (acs-engine)?

## Azure Container Service Engine

Azure Container Service Engine ([ACS Engine](https://github.com/Azure/acs-engine)) is what Azure Container Service uses when it deploys on Azure. Microsoft [announced](https://azure.microsoft.com/en-us/blog/azure-container-service-the-cloud-s-most-open-option-for-containers/) the availability of it's source code available as a project on [github](https://github.com/Azure/acs-engine) on November 7th, 2016.

With `acs-engine` is a good option if you need to customize a deployment scenarios where ACS currently does not cover. Examples here are when deploying to an existing or custom [VNet](https://github.com/Azure/acs-engine/blob/master/examples/vnet) or deploying a very large cluster - see this example of how [ACS-Engine enables you to create customized Docker enabled cluster on Microsoft Azure with 1200 nodes](https://github.com/Azure/acs-engine/tree/master/examples/largeclusters). 

ACS-Engine reads a cluster definition as it input and it will output an ARM (Azure Resource Manager) template that can then be deployed on Azure. An added benefit from having your Kubernetes cluster defined as an ARM template is that it can be checked into a source control repository.

> ACS Engine essentially allows you to modify or customize a deployment.

## Value Proposition


Solution | Value Proposition | Orchestrators
--- | --- | ---
ACS | Turn-key solution to deploy containers on Azure.| Kubernetes, DC/OS, Swarm.
ACS-Engine | Option if you need to make customizations to the deployment beyond what the Azure Container Service officially supports.  | Kubernetes, DC/OS, Swarm.

## References 


* [ACS-Engine on Github](https://github.com/Azure/acs-engine)
* [ACS-Engine examples and walkthroughs](https://github.com/Azure/acs-engine/tree/master/examples)
* [ACS-Engine Kubernetes Walkthrough](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes.md)
* [Microsoft Azure Container Service - Kubernetes Walkthrough](https://docs.microsoft.com/en-us/azure/container-service/kubernetes/container-service-kubernetes-walkthrough)

