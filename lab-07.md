# Lab 07 - L4 Load Balancer and L7 Load Balancer

In this lab, we use [Qwiklabs](https://www.qwiklabs.com/).

Qwiklabs Lab: [Orchestrating the Cloud with Kubernetes](https://google.qwiklabs.com/focuses/557?locale=en&parent=catalog)

We will:
- Create a Google Kubernetes Engine Cluster.
- Quick demo
- Create a Pod
- Create a service (NodePort)
- Add Label
- Deploy a application

# L4 Load Balancer

## Deploy application

Deploy nginx deployment.

```
kubectl create deployment helloweb --image=nginx
```

Revese a static ip

```
gcloud compute addresses create helloweb-ip --region us-central1
```

Get IP.

```
gcloud compute addresses describe helloweb-ip --region us-central1 | grep address
```

Create a new file `svc-lb.yaml`, change `REVERSE_STATIC_IP` to above IP.

```
apiVersion: v1
kind: Service
metadata:
  name: helloweb-lb
spec:
  type: LoadBalancer
  loadBalancerIP: REVERSE_STATIC_IP
  selector:
    app: helloweb
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

Deploy service.

```
kubectl apply -f svc-lb.yaml
```

> Check Cloud Console

- Kubernetes Engine -> Services & Ingress
- Network services

## Internal Load Balancer

Create a new file `svc-ilb.yaml`, add annotation `cloud.google.com/load-balancer-type: "Internal"`.

```
apiVersion: v1
kind: Service
metadata:
  name: helloweb-ilb
  annotations:
    cloud.google.com/load-balancer-type: "Internal"
spec:
  type: LoadBalancer
  loadBalancerIP: 10.128.0.50
  selector:
    app: helloweb
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

Deploy service.

```
kubectl apply -f svc-ilb.yaml
```

> Check Cloud Console

- Kubernetes Engine -> Services & Ingress
- Network services

> Create a VM to send http request to `10.128.0.50`.

```
curl 10.128.0.50
```

We should get output like:

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## Clean

```
gcloud compute addresses delete helloweb-ip --region us-central1

kubectl delete -f svc-lb.yaml
kubectl delete -f svc-ilb.yaml
kubectl delete deploy helloweb
```

# L7 Load Balancer

## Create Service-01

```
kubectl create deployment service-01 --image=nginx
```

After pod is ready, update the `index.html`

```
kubectl exec $(kubectl get po -l app=service-01 -o jsonpath='{.items[0].metadata.name}') \
  -- sh -c 'echo "Here is Service-01" > /usr/share/nginx/html/index.html'

kubectl exec $(kubectl get po -l app=service-01 -o jsonpath='{.items[0].metadata.name}') \
  -- sh -c 'mkdir -p /usr/share/nginx/html/app;echo "Here is Service-01 App" > /usr/share/nginx/html/app/index.html'
```

Expose Service-01 with NodePort service

```
kubectl expose deployment service-01 --port=80 --type=NodePort
```

## Create Service-02

```
kubectl create deployment service-02 --image=nginx
```

After pod is ready, update the `index.html`

```
kubectl exec $(kubectl get po -l app=service-02 -o jsonpath='{.items[0].metadata.name}') \
  -- sh -c 'echo "Here is Service-02" > /usr/share/nginx/html/index.html'

kubectl exec $(kubectl get po -l app=service-02 -o jsonpath='{.items[0].metadata.name}') \
  -- sh -c 'mkdir -p /usr/share/nginx/html/forum;echo "Here is Service-02 Forum" > /usr/share/nginx/html/forum/index.html'
```

Expose Service-02 with NodePort service

```
kubectl expose deployment service-02 --port=80 --type=NodePort
```

## Deploy Ingress

Revese a static ip

```
gcloud compute addresses create helloweb-ip --global
```

Create a new file `ingress.yaml`

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: helloweb
  annotations:
    kubernetes.io/ingress.global-static-ip-name: helloweb-ip
spec:
  backend:
    serviceName: service-01
    servicePort: 80
```

Create Ingress

```
kubectl apply -f ingress.yaml
```

> Check Cloud Console

- Kubernetes Engine -> Services & Ingress
- Network services

Wait for 5 minutes, send http request to INGRESS IP.

```
curl INGRESS_IP
```

We should get output like:

```
Here is Service-01
```

## Routing with path

Edit `ingress.yaml`

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: helloweb
  annotations:
    kubernetes.io/ingress.global-static-ip-name: helloweb-ip
spec:
  rules:
  - http:
     paths:
      - path: /app/*
        backend:
          serviceName: service-01
          servicePort: 80
      - path: /forum/*
        backend:
          serviceName: service-02
          servicePort: 80
```

Create Ingress

```
kubectl apply -f ingress.yaml
```

> Check Cloud Console

- Kubernetes Engine -> Services & Ingress
- Network services

Wait for 5 minutes, send http request to INGRESS IP.

```
curl INGRESS_IP/app/
curl INGRESS_IP/form/
```

We should get output like:

```
Here is Service-01 App
Here is Service-02 Forum
```

## Routing with host

Edit `ingress.yaml`

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: helloweb
  annotations:
    kubernetes.io/ingress.global-static-ip-name: helloweb-ip
spec:
  rules:
  - host: app.foo.dev
    http:
     paths:
      - path: /*
        backend:
          serviceName: service-01
          servicePort: 80
  - host: forum.foo.dev
    http:
     paths:
      - path: /*
        backend:
          serviceName: service-02
          servicePort: 80
```

Create Ingress

```
kubectl apply -f ingress.yaml
```

Test

```
curl -H "Host: app.foo.dev" 34.107.234.199
curl -H "Host: forum.foo.dev" 34.107.234.199
```

We should get output like:

```
Here is Service-01
Here is Service-02
```

## Clean

```
kubectl delete ingress.yaml
kubectl delete services service-01
kubectl delete services service-02
kubectl delete deployment service-01
kubectl delete deployment service-02
```
