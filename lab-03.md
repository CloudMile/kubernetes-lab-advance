# Lab 03 - Using Statefulset

Edit 'statefulset.yaml'

```
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

```
kubectl apply -f statefulset.yaml
```

Check status.

```
kubectl get statefulset
kubectl get pods
kubectl get pvc,pv
```

## Scale

```
kubectl scale statefulset web --replicas=3
```

Check status.

```
kubectl get statefulset
kubectl get pods
kubectl get pvc,pv
```

```
kubectl scale statefulset web --replicas=2
```

Check status.

```
kubectl get statefulset
kubectl get pods
kubectl get pvc,pv
```

## Headless service

Edit `svc-headless.yaml`

```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
```

Check the DNS records.

```
kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm /bin/sh

nslookup nginx
nslookup web-0.nginx
nslookup web-1.nginx
```

## Delete pods

In other terminal:

```
kubectl delete pod -l app=nginx
```

Check the DNS records.

```
kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm /bin/sh

nslookup nginx
nslookup web-0.nginx
nslookup web-1.nginx
```

## Clean

```
kubectl delete -f svc-headless.yaml
kubectl delete -f statefulset.yaml
```
