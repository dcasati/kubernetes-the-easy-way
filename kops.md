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
