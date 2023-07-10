# Ingress with ssl/tls
https://snyk.io/blog/setting-up-ssl-tls-for-kubernetes-ingress/

## 1- Install nginx ingress controller

```bash
NAMESPACE=ingress-nginx

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --create-namespace \
  --namespace $NAMESPACE \
  --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"=/healthz

kubectl --namespace ingress-nginx get services -o wide -w ingress-nginx-controller
```

## 2- Create a certificate

```bash
mkdir certs

openssl req -x509 -nodes -days 9999 -newkey rsa:2048 -keyout certs/ingress-tls.key -out certs/ingress-tls.crt
```
Enter the organizational information to be integrated into the certificate request, such as the organization name, country code, location, and email address. Note that this demonstration uses www.ingress-tls.com as the certificate's fully qualified domain name (FQDN) name. 

akspocrr.com

## 3- Create a k8s secret

```bash
kubectl create secret tls ingress-cert --namespace dev --key=certs/ingress-tls.key --cert=certs/ingress-tls.crt -o yaml
```

## 4- Create an ingress resource

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aks-helloworld  
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aks-helloworld
  template:
    metadata:
      labels:
        app: aks-helloworld
    spec:
      containers:
      - name: aks-helloworld
        image: mcr.microsoft.com/azuredocs/aks-helloworld:v1
        ports:
        - containerPort: 80
        env:
        - name: TITLE
          value: "Welcome to Azure Kubernetes Service (AKS)"
---
apiVersion: v1
kind: Service
metadata:
  name: aks-helloworld
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: aks-helloworld
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: aks-helloworld
spec:
  ingressClassName: nginx
  rules:
  - host: akspocrr.com
    http:
      paths:
      - backend:
          service:
            name: aks-helloworld
            port:
              number: 80
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - akspocrr.com
    secretName: ingress-cert
```
## 5- Set on host file the ip and akspocrr

 C:\Windows\System32\Drivers\Etc\Hosts

ip akspocrr.com

## 6- Test

```bash
curl -k https://akspocrr.com

echo "http://hello.${SERVICE_IP}.nip.io"
```