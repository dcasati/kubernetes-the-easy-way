# KOPS

## Creating a cluster

Export the NAME and the s3 state store to Kops.

```bash
export NAME=demo.cross-cloud.xyz   
export KOPS_STATE_STORE=s3://demo-cross-cloud-xyz-state-store
```
Then proceed by creating a cluster.
```bash
kops create cluster --yes \
  --zones=us-west-2a cross-cloud.xyz 
  --topology private \
  --dns-zone  ZJ57ASEDGKT8B 
  --networking calico \
  --bastion
```

### Cleaning up 

To Delete a cluster

```bash
kops delete cluster --name cross-cloud.xyz --yes
```
## Exporting kubecfg

```bash
kops export  kubecfg --name cross-cloud.xyz
```

## Working with multiple contexts

### What contexts are available?
```bash 
kubectl config get-contexts
```

> Output
```
CURRENT   NAME                             CLUSTER                          AUTHINFO                               NAMESPACE
          azure-k8s-cross-cloud-5a1dae37   azure-k8s-cross-cloud-5a1dae37   azure-k8s-cross-cloud-5a1dae37-admin   
*         cross-cloud.xyz                  cross-cloud.xyz                  cross-cloud.xyz    
```
### Change the current context

```bash
kubectl config set current-context  azure-k8s-cross-cloud-5a1dae37
```

