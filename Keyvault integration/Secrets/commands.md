### 1- enable addon
```	
az aks enable-addons --addons azure-keyvault-secrets-provider --name myAKSCluster --resource-group myResourceGroup
```	

verify the addon is enabled
```	
kubectl get pods -n kube-system -l 'app in (secrets-store-csi-driver,secrets-store-provider-azure)'
```	

### 2- Create secret
```	
az keyvault secret set --vault-name my-kv-rr -n dbConnection --value database-sample:5600
```	

## Enable autorotation on an existing AKS cluster

####  Update an existing cluster to enable autorotation of secrets using the az aks addon update command and the enable-secret-rotation parameter.
```	
az aks addon update -g myResourceGroup -n myAKSCluster2 -a azure-keyvault-secrets-provider --enable-secret-rotation
```	

Specify a custom rotation interval using the rotation-poll-interval parameter.
```	
az aks addon update -g myResourceGroup -n myAKSCluster2 -a azure-keyvault-secrets-provider --enable-secret-rotation --rotation-poll-interval 5m
```

disable the addon
```
az aks addon disable -g myResourceGroup -n myAKSCluster2 -a azure-keyvault-secrets-provider
```

re-enable the addon without the `enable-secret-rotation` parameter
```
az aks addon enable -g myResourceGroup -n myAKSCluster2 -a azure-keyvault-secrets-pr
```

### 3 - run the following command to create a pod that uses the secret
```
kubectl apply -f 1-dep-kv-secret.yaml
```

### 4 - verify the secret is mounted
```
kubectl exec -it my-app -- ls /mnt/secrets-store
```