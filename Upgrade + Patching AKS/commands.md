## Talk about automatic patches 

## Remove usage of deprecated APIs (recommended)
In the Azure portal, navigate to your cluster's overview page, and select Diagnose and solve problems.

Navigate to the Known Issues, Availability and Performance category, and select Selected Kubernetes API deprecations.

```
bypass derecations: az aks update --name myAKSCluster --resource-group myResourceGroup --upgrade-settings IgnoreKubernetesDeprecations --upgrade-override-until 2023-04-01T13:00:00Z
```

### 1 - Set max surge values
Set max surge for a new node pool
az aks nodepool add -n mynodepool -g MyResourceGroup --cluster-name MyManagedCluster --max-surge 33%

Update max surge for an existing node pool 
az aks nodepool update -n mynodepool -g MyResourceGroup --cluster-name MyManagedCluster --max-surge 5

### 2 - Show available versions
 az aks get-upgrades --resource-group myResourceGroup --name myAKSCluster --output table

### 3 - Cordon & Drain nodes
kubectl get nodes

kubectl cordon aks-nodepool1-31721111-vmss000000 aks-nodepool1-31721111-vmss000001 aks-nodepool1-31721111-vmss000002

kubectl drain aks-nodepool1-31721111-vmss000000 aks-nodepool1-31721111-vmss000001 aks-nodepool1-31721111-vmss000002 --ignore-daemonsets --delete-emptydir-data

kubectl get pods -o wide -A

### 4 - Upgrade cluster

az aks upgrade \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --kubernetes-version KUBERNETES_VERSION

### view upgrade events
kubectl get events

### show new version
az aks show --resource-group myResourceGroup --name myAKSCluster --output table

### Patching nodes

check the current node image versions of the nodes in a cluster:
az aks nodepool list \
   --resource-group <ResourceGroupName> --cluster-name <AKSClusterName> \
   --query "[].{Name:name,NodeImageVersion:nodeImageVersion}" --output table

to find out the latest available node image version:
az aks nodepool get-upgrades \
   --resource-group <ResourceGroupName> --cluster-name <AKSClusterName> \
   --nodepool-name <NodePoolName> --output table

az aks nodepool show \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --query nodeImageVersion

az aks upgrade \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-image-only

To update the OS image of a node pool without doing a Kubernetes cluster upgrade
az aks nodepool upgrade \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --node-image-only

Upgrade node images with node surge
az aks nodepool update \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --max-surge 33% \
    --no-wait

kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.labels.kubernetes\.azure\.com\/node-image-version}{"\n"}{end}'

