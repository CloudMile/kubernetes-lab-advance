# Lab 02 - Using Daemonset

Edit `ds.yaml`

```
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: ds-one
spec:
  template:
    metadata:
      labels:
        system: DaemonSetOne
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

```
kubectl apply -f ds.yaml
```

Check Deamon.

```
kubectl get ds
```

Check number of pod.

```
kubectl get pods
```

Check labels of node

```
kubectl get nodes --show-labels
```

## Specific Node

Edit `ds.yaml`

```
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: ds-one
spec:
  template:
    metadata:
      labels:
        system: DaemonSetOne
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
      nodeSelector:
        test: "true"
```

```
kubectl apply -f ds.yaml
```

Check Deamon.

```
kubectl get ds
```

Check number of pod.

```
kubectl get pods
```

Add label to node

```
kubectl label node minikube test=true
```

Check Deamon.

```
kubectl get ds
```

Check number of pod.

```
kubectl get pods
```

## Clean

```
kubectl label node minikube test-
kubectl delete -f ds.yaml
```
