# ACS and ACS-Engine - What is it all about ?

## Azure Container Service

According to it's [definition](https://docs.microsoft.com/en-us/azure/container-service/): 

> Azure Container Service allows you to quickly deploy a production ready Kubernetes, DC/OS, or Docker Swarm cluster.

In other words, Azure Container Service (ACS) is essentially a turn-key solution that allow the deployment of containers with various orchestration engines. Take Kubernetes as an example. ACS takes away all of the deployment, upgrade and management tasks that are part of spinning up a Kubernetes cluster such as certificates and PKI creation and distribution to the controllers and worker nodes.

**Key takeaway:**
> ACS is a turn-key solution to quickly deploy containers on Azure.

So then, what is Azure Container Service Engine (acs-engine)?

## Azure Container Service Engine

Azure Container Service Engine ([ACS Engine](https://github.com/Azure/acs-engine)) is what Azure Container Service uses when it deploys on Azure. Microsoft [announced](https://azure.microsoft.com/en-us/blog/azure-container-service-the-cloud-s-most-open-option-for-containers/) the availability of it's source code available as a project on [github](https://github.com/Azure/acs-engine) on November 7th, 2016.

With `acs-engine` is a good option if you need to customize a deployment scenarios where ACS currently does not cover. Examples here are when deploying to an existing or custom [VNet](https://github.com/Azure/acs-engine/blob/master/examples/vnet) or deploying a very large cluster - see this example of how [ACS-Engine enables you to create customized Docker enabled cluster on Microsoft Azure with 1200 nodes](https://github.com/Azure/acs-engine/tree/master/examples/largeclusters). 

ACS-Engine reads a cluster definition as it input and it will output an ARM (Azure Resource Manager) template that can then be deployed on Azure. An added benefit from having your Kubernetes cluster defined as an ARM template is that it can be checked into a source control repository.

**NOTE:**
> A key difference between ACS Engine and ACS is that *ACS Engine* will create a _cluster_ and not a resource managed by ACS - ACS is unaware of them. In practical terms that means that you will not see anything created by ACS Engine when you run `az acs list`.

**Key takeaway:**
> ACS Engine essentially allows you to modify or customize a deployment.

## Getting Started with ACS-Engine

Here's an example of how to use `acs-engine`. For this walkthrough, we will be using the [kubernetes.json](https://github.com/Azure/acs-engine/blob/master/examples/kubernetes.json) example. This will deploy one master and two Linux agents.

1. Install the `acs-engine`.

    1. Fetch the latest version of `acs-engine` from [here](https://github.com/Azure/acs-engine/releases/latest)

    ```
    $ wget https://github.com/Azure/acs-engine/releases/download/v0.8.0/acs-engine-v0.8.0-linux-amd64.tar.gz
    ```
    2. Unpack the tarball:
    ```
    $ tar xvzf acs-engine-v0.8.0-linux-amd64.tar.gz
    ```
    3. Move the binary:
    ```
    $ sudo mv acs-engine-v0.8.0-linux-amd64/acs-engine /usr/local/bin
    ```
    > Note: For other method of installation, including compiling from the source please refer to this [guide.](https://github.com/Azure/acs-engine/blob/master/docs/acsengine.md#install-acs-engine)

2. Gather Information
    
    1. Select a subscription to use with `acs-engine`

    ```
    $ AZURE_SUBSCRIPTION=$(az account list --query "[][id]" -o tsv)
    ```
    > Note: If you have more than one subscription you can pick an choose them by running `az account list`

    2. Define a `DNS_PREFIX` and a `LOCATION` to provision the cluster. It's important to note that the `DNS_PREFIX` must be unique.

    ```
    $ DNS_PREFIX=pistachio-cannoli
    $ LOCATION=westus2
    ```

    Here's a summary of the parameters:

    Parameter | Value
    --- | --- 
    AZURE_SUBSCRIPTION | XXXXXXXXXXX
    DNS_PREFIX | pistachio-cannoli
    LOCATION | westus2

    3. Create the `kubernetes.json` file:

```
cat > kubernetes.json << EOF
{
  "apiVersion": "vlabs",
  "properties": {
    "orchestratorProfile": {
      "orchestratorType": "Kubernetes"
    },
    "masterProfile": {
      "count": 1,
      "dnsPrefix": "",
      "vmSize": "Standard_D2_v2"
    },
    "agentPoolProfiles": [
      {
        "name": "agentpool1",
        "count": 3,
        "vmSize": "Standard_D2_v2",
        "availabilityProfile": "AvailabilitySet"
      }
    ],
    "linuxProfile": {
      "adminUsername": "azureuser",
      "ssh": {
        "publicKeys": [
          {
            "keyData": ""
          }
        ]
      }
    },
    "servicePrincipalProfile": {
      "clientId": "",
      "secret": ""
    }
  }
}
EOF
```

3. Deploy

    1. With the information above, execute `acs-engine deploy`:

    ```
    $ acs-engine deploy --subscription-id $AZURE_SUBSCRIPTION \
      --dns-prefix $DNS_PREFIX 
      --location $LOCATION 
      --auto-suffix 
      --api-model kubernetes.json

    WARN[0000] To sign in, use a web browser to open the page https://aka.ms/devicelogin and enter the code EBG9AC4ND to authenticate. 
    WARN[0125] apimodel: missing masterProfile.dnsPrefix will use "pistachio-cannoli-59deb526" 
    WARN[0125] --resource-group was not specified. Using the DNS prefix from the apimodel as the resource group name: pistachio-cannoli-59deb526 
    WARN[0127] apimodel: ServicePrincipalProfile was missing or empty, creating application... 
    WARN[0128] created application with applicationID (360a8f78-3f88-4d84-951e-9229f91920d4) and servicePrincipalObjectID (acef65d4-9169-4940-b4ad-d5a67283fb4d). 
    WARN[0128] apimodel: ServicePrincipalProfile was empty, assigning role to application... 
    INFO[0149] Starting ARM Deployment (pistachio-cannoli-59deb526-114104692). This will take some time... 
    INFO[0582] Finished ARM Deployment (pistachio-cannoli-59deb526-114104692). 
    ```
    `acs-engine` will populate the `_output` directory with all of the files necessary to deploy the cluster, including various `kubeconfig` files for all of the supported regions.

    ```
    _output
    └── pistachio-cannoli-59deb526
        ├── apimodel.json
        ├── apiserver.crt
        ├── apiserver.key
        ├── azuredeploy.json
        ├── azuredeploy.parameters.json
        ├── azureuser_rsa
        ├── ca.crt
        ├── ca.key
        ├── client.crt
        ├── client.key
        ├── kubeconfig
        │   ├── kubeconfig.australiaeast.json
        │   ├── kubeconfig.australiasoutheast.json
        │   ├── kubeconfig.brazilsouth.json
        │   ├── kubeconfig.canadacentral.json
        │   ├── kubeconfig.canadaeast.json
        │   ├── kubeconfig.centralindia.json
        │   ├── kubeconfig.centraluseuap.json
        │   ├── kubeconfig.centralus.json
        │   ├── kubeconfig.chinaeast.json
        │   ├── kubeconfig.chinanorth.json
        │   ├── kubeconfig.eastasia.json
        │   ├── kubeconfig.eastus2euap.json
        │   ├── kubeconfig.eastus2.json
        │   ├── kubeconfig.eastus.json
        │   ├── kubeconfig.germanycentral.json
        │   ├── kubeconfig.germanynortheast.json
        │   ├── kubeconfig.japaneast.json
        │   ├── kubeconfig.japanwest.json
        │   ├── kubeconfig.koreacentral.json
        │   ├── kubeconfig.koreasouth.json
        │   ├── kubeconfig.northcentralus.json
        │   ├── kubeconfig.northeurope.json
        │   ├── kubeconfig.southcentralus.json
        │   ├── kubeconfig.southeastasia.json
        │   ├── kubeconfig.southindia.json
        │   ├── kubeconfig.uksouth.json
        │   ├── kubeconfig.ukwest.json
        │   ├── kubeconfig.usgoviowa.json
        │   ├── kubeconfig.usgovvirginia.json
        │   ├── kubeconfig.westcentralus.json
        │   ├── kubeconfig.westeurope.json
        │   ├── kubeconfig.westindia.json
        │   ├── kubeconfig.westus2.json
        │   └── kubeconfig.westus.json
        ├── kubectlClient.crt
        └── kubectlClient.key
    ```
3. Connect to the cluster
    
    1. Using the `kubeconfig` for our location (westus2), connect to the cluster with `kubectl`:

    ```
    $ KUBECONFIG=_output/pistachio-cannoli-59deb526/kubeconfig/kubeconfig.westus2.json
    $ kubectl cluster-info

    Kubernetes master is running at https://pistachio-cannoli-59deb526.westus2.cloudapp.azure.com
    Heapster is running at https://pistachio-cannoli-59deb526.westus2.cloudapp.azure.com/api/v1/namespaces/kube-system/services/heapster/proxy
    KubeDNS is running at https://pistachio-cannoli-59deb526.westus2.cloudapp.azure.com/api/v1/namespaces/kube-system/services/kube-dns/proxy
    kubernetes-dashboard is running at https://pistachio-cannoli-59deb526.westus2.cloudapp.azure.com/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy
    tiller-deploy is running at https://pistachio-cannoli-59deb526.westus2.cloudapp.azure.com/api/v1/namespaces/kube-system/services/tiller-deploy/proxy

    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
    ```
    2. Get information about the nodes:

    ```
    $ kubectl get nodes
    NAME                        STATUS    ROLES     AGE       VERSION
    k8s-agentpool1-27471561-0   Ready     agent     8m        v1.7.7
    k8s-agentpool1-27471561-1   Ready     agent     8m        v1.7.7
    k8s-agentpool1-27471561-2   Ready     agent     8m        v1.7.7
    k8s-master-27471561-0       Ready     master    8m        v1.7.7
    ```
That's it ! your Kubernetes cluster is now up and running. 

### Features and More Advance Examples

1. [Managed Identity](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/features.md#managed-identity)
1. [Managed Disks (Persistent Volumes)](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/features.md#managed-disks)
1. [Network Policy with Calico](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/features.md#network-policy-enforcement-with-calico)
1. [Custom VNet](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/features.md#custom-vnet)
1. [Using Azure integrated networking (CNI)](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/features.md#using-azure-integrated-networking-cni)

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

