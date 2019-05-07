# Lab 08 - L7 Load Balancer

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