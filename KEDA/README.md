# Keda Lab on AKS

## Keda Architecture Diagram:


![Keda Architecture](https://keda.sh/img/keda-arch.png)

## Keda Labs Setup:

### 1- Install a demo app with LoadBalancer service exposed
```
$ kubectl apply -f deployment.yaml
```
### 2- Install nginx if not in the cluster already 
```	
$ NAMESPACE=ingress-basic

$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

$ helm install ingress-nginx ingress-nginx/ingress-nginx \
    --create-namespace \
    --namespace $NAMESPACE \
    --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"=/healthz
```

### 3- Route traffic to our app using ingress route
```
$ kubectl apply -f ingress.yaml
```

### 4- Deploy Prometheus
```
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```
```
$ helm install prometheus prometheus-community/prometheus \ 
--set server.service.type=NodePort --set server.service.LoadBalancer
```

Verify that is running 
```
$ kubectl get pods
```

Go to the webpage to hit prometheus server https://$PrometheusLoadBalancerIP

![Prometheus Server](https://www.nginx.com/wp-content/uploads/2021/09/Prometheus_homepage-640x227.png)

See the graph for nginx connections active
```
$ sum(avg_over_time(nginx_ingress_nginx_connections_active{app_kubernetes_io_instance="main"}[1m]))
```

### 5- Deploy Keda add-on on AKS Cluster
```
$ az extension add --name aks-preview
```
```
$ az extension update --name aks-preview
```
### Register the 'AKS-KedaPreview' feature flag
```
$ az feature register --namespace "Microsoft.ContainerService" --name "AKS-KedaPreview"

$ az feature show --namespace "Microsoft.ContainerService" --name "AKS-KedaPreview"

When the status reflects Registered, refresh the registration of the Microsoft.ContainerService resource provider by using the az provider register command:

$ az provider register --namespace Microsoft.ContainerService
```
### Install the KEDA add-on with Azure CLI
```
$ az group create --name myResourceGroup --location westeurope

$ az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --enable-keda

Get the credentials of the cluster:
$ az aks get-credentials --resource-group myResourceGroup --name myAKSCluster

Verify Keda:
$ az aks show -g "myResourceGroup" --name myAKSCluster --query "workloadAutoScalerProfile.keda.enabled"

$ kubectl get pods -n kube-system
keda-operator-********-k5rfv                     1/1     Running   0          43m
keda-operator-metrics-apiserver-*******-sj857   1/1     Running   0          43m
```

### 6- Deploy Keda ScaledObject based on Prometheus metrics (nginx connections active)
```
$ kubectl apply -f scaledobject.yaml
```

### 7- Create a load test to generate traffic using Azure Load Test
```
Create a load test using azure service and point to the ingress url of the app
```

### 8 - Verify that the pods are scaled up and down based on the load test on Grafana using the Dashboard json file
```
$ dasboard.json
```

### Scale external cosmosDB

https://github.com/Azure-Samples/cosmos-aks-keda

### sources:

- https://keda.sh/
- https://learn.microsoft.com/pt-pt/azure/aks/keda-about
- https://learn.microsoft.com/en-us/azure/aks/keda-deploy-add-on-cli#install-the-keda-add-on-with-azure-cli
- https://azure.microsoft.com/en-us/products/load-testing/