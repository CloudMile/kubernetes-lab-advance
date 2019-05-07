# Lab 07 - L4 Load Balancer

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

Revese a static ip

```
gcloud compute addresses create helloweb-ip --region asia-east1
```

Get ip

```
gcloud compute addresses describe helloweb-ip --region asia-east1 | grep address
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
  type: LoadBalancer
  loadBalancerIP: REVERSE_STATIC_IP
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

> Check Cloud Console

## Internal Load Balancer

Edit `svc-lb.yaml`

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  annotations:
    cloud.google.com/load-balancer-type: "Internal"
  labels:
    run: nginx
spec:
  type: LoadBalancer
  loadBalancerIP: 10.140.0.50
  selector:
    run: nginx
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

> Create a VM to test.

## Clean

```
gcloud compute addresses delete helloweb-ip --region asia-east1
kubectl delete -f svc-lb.yaml
kubectl delete -f deploy.yaml
```