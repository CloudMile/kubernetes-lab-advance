# Lab 08 - L7 Load Balancer

# Minikube

## Create Service-01

```
kubectl run service-01 --image=nginx
```

After pod is ready, update the `index.html`

```
kubectl exec $(kubectl get po -l run=service-01 -o jsonpath='{.items[0].metadata.name}') \
  -- sh -c 'echo "Here is Service-01" > /usr/share/nginx/html/index.html'
```

Expose Service-01 with NodePort service

```
kubectl expose deployment service-01 --port=80 --type=NodePort
```


## Create Service-02

```
kubectl run service-02 --image=nginx
```

After pod is ready, update the `index.html`

```
kubectl exec $(kubectl get po -l run=service-02 -o jsonpath='{.items[0].metadata.name}') \
  -- sh -c 'echo "Here is Service-02" > /usr/share/nginx/html/index.html'
```

Expose Service-02 with NodePort service

```
kubectl expose deployment service-02 --port=80 --type=NodePort
```

## Enable Ingress addon

Check addons status

```
minikube addons list
```

```
minikube addons enable ingress
```

Edit `ingress.yaml`

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  backend:
    serviceName: service-01
    servicePort: 80
```

Create Ingress

```
kubectl apply -f ingress.yaml
```

Show Ingress

```
kubectl get ingress
```

In short

```
kubectl get ing
```

See detail

```
kubectl describe ingress example-ingress
```

Test

```
curl $(minikube ip)
```

## Routing with path

Edit `ingress.yaml`

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
     paths:
      - path: /app
        backend:
          serviceName: service-01
          servicePort: 80
      - path: /forum
        backend:
          serviceName: service-02
          servicePort: 80
```

Create Ingress

```
kubectl apply -f ingress.yaml
```

See detail

```
kubectl describe ingress example-ingress
```

Test

```
curl $(minikube ip)/app
curl $(minikube ip)/forum
```

## Routing with host

Edit `ingress.yaml`

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
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

See detail

```
kubectl describe ingress example-ingress
```

Edit `/etc/hosts`, add two record. Remember change the `MINIKUBE_IP`

```
MINIKUBE_IP app.foo.dev
MINIKUBE_IP forum.foo.dev
```

Test

```
curl app.foo.dev
curl forum.foo.dev
```

## Support HTTPS

__Create a TLS certificate for app.foo.dev__

```
DOMAIN=app.foo.dev
openssl genrsa -out $DOMAIN.key 2048
openssl req -new -key $DOMAIN.key -out $DOMAIN.csr \
  -subj "/CN=$DOMAIN"
openssl x509 -req -days 365 -in $DOMAIN.csr -signkey $DOMAIN.key \
    -out $DOMAIN.crt
```

Create secret

```
kubectl create secret tls app-tls \
  --cert=$DOMAIN.crt \
  --key=$DOMAIN.key
```

__Create a TLS certificate for forum.foo.dev__

```
DOMAIN=forum.foo.dev
openssl genrsa -out $DOMAIN.key 2048
openssl req -new -key $DOMAIN.key -out $DOMAIN.csr \
  -subj "/CN=$DOMAIN"
openssl x509 -req -days 365 -in $DOMAIN.csr -signkey $DOMAIN.key \
    -out $DOMAIN.crt
```

Create secret

```
kubectl create secret tls forum-tls \
  --cert=$DOMAIN.crt \
  --key=$DOMAIN.key
```

__Ingress with HTTPS__

Edit `ingress.yaml`

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - secretName: app-tls
  - secretName: forum-tls
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

See detail

```
kubectl describe ingress example-ingress
```

Test

```
curl -k https://app.foo.dev
curl -k https://forum.foo.dev
```

## Clean

```
kubectl delete ingress.yaml
kubectl delete secret app-tls
kubectl delete secret forum-tls
kubectl delete services service-01
kubectl delete services service-02
kubectl delete deployment service-01
kubectl delete deployment service-02
```

Check `/etc/hosts`

----

# GKE

Edit `deploy.yaml`

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
```

```
kubectl apply -f deploy.yaml
```

Edit `svc-lb.yaml`

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    run: nginx
spec:
  type: NodePort
  selector:
    run: nginx
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

```
kubectl apply -f svc-lb.yaml
```

Revese a static ip

```
gcloud compute addresses create helloweb-ip --global
```

Edit `ing.yaml`

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
      - path: /*
        backend:
          serviceName: nginx-svc
          servicePort: 80
```

```
kubectl apply -f ing.yaml
```

> Wait 5 minutes


```
kubectl get ing
```

> Test the `EXTERNAL-IP`


### Set SSL certificate

Create a TLS certificate

```
DOMAIN=helloweb.dev
openssl genrsa -out $DOMAIN.key 2048
openssl req -new -key $DOMAIN.key -out $DOMAIN.csr \
  -subj "/CN=$DOMAIN"
openssl x509 -req -days 365 -in $DOMAIN.csr -signkey $DOMAIN.key \
    -out $DOMAIN.crt
```

Upload certificate to Project

```
gcloud compute ssl-certificates create helloweb-certificate \
  --certificate $DOMAIN.crt \
  --private-key $DOMAIN.key
```

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: helloweb
  annotations:
    kubernetes.io/ingress.global-static-ip-name: helloweb-ip
    ingress.gcp.kubernetes.io/pre-shared-cert: helloweb-certificate
spec:
  rules:
  - host: helloweb.dev
    http:
      paths:
      - backend:
          serviceName: nginx-svc
          servicePort: 80
```

```
kubectl apply -f ing.yaml
```

> Wait 5 minutes

```
kubectl get ingress
```

Add name resolv localy.

```
sudo vi /etc/hosts
```

Test

```
curl -kv https://helloweb.dev
```

## Clean

```
kubectl delete -f ing.yaml
kubectl delete -f svc-lb.yaml
kubectl delete -f deploy.yaml
gcloud compute ssl-certificates delete helloweb-certificate
gcloud compute addresses delete helloweb-ip --global
```