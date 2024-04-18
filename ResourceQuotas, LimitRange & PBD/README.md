# Pod Disruption Budgets


1. Create a sample deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - name: http
    port: 80
    targetPort: 80
```
```PowerShell
kubectl apply -f pdb.yml
```

5. Create a Pod Disruption Budget

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: nginx-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: nginx
```

```PowerShell
kubectl apply -f pdb-policy.yml
```

This PDB ensures that at least two replica of the nginx deployment is available at all times. If a node goes down or a pod is evicted, Kubernetes will attempt to reschedule the pod on a different node, but will not evict any pods that would violate the PDB.

Once the resources are created, you can test the PDB by manually evicting one of the nginx pods:



```PowerShell
kubectl cordon nodes

kubectl drain  aks-systemnp-90583515-vmss000003 --ignore-daemonsets --delete-emptydir-data
```

It will raise an error and scale another node if autoscaler enable, otherwise wont let you drain the node since it will violate the PDB policy.

## Delete all
```PowerShell
kubectl delete -f pdb-policy.yml

kubectl delete -f pdb.yml
```
