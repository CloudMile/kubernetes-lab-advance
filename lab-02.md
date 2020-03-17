# Lab 02 - Using Daemonset

We use [KataCota](https://www.katacoda.com/) as playground.

This lab is on the environment: [Deploy Containers Using YAML](https://www.katacoda.com/courses/kubernetes/creating-kubernetes-yaml-definitions).

Click "START SCENARIO". We will have __One Hour__ lab environment. After __Onre Hour__, we have to refresh browser to get new environment.

The environment provide We need the editor and terminal. We need to wait the minibuke ready.

```
minikube start --wait=false
$
$ minikube start --wait=false
* minikube v1.8.1 on Ubuntu 18.04
* Using the none driver based on user configuration
* Running on localhost (CPUs=2, Memory=2460MB, Disk=145651MB) ...
* OS release is Ubuntu 18.04.4 LTS
* Preparing Kubernetes v1.17.3 on Docker 19.03.6 ...
  - kubelet.resolv-conf=/run/systemd/resolve/resolv.conf
* Launching Kubernetes ...
* Enabling addons: default-storageclass, storage-provisioner
* Configuring local host environment ...
* Done! kubectl is now configured to use "minikube"
```

The lab environemnt screenshot:

![](katacoda/01.png)


We can click mouse right on the `/root` to create new files:

![](katacoda/02.png)


## Deploy Daemonset

Create a new file `ds.yaml`

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-one
spec:
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
        image: nginx
        ports:
        - containerPort: 80
```

Deploy daemonset.

```
kubectl apply -f ds.yaml
```

Check Deamon.

```
kubectl get daemonset
```

We should get output like:

```
NAME     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
ds-one   1         1         1       1            1           <none>          79s
```

Check number of pod.

```
kubectl get pods
```

We should get output like:

```
NAME           READY   STATUS    RESTARTS   AGE
ds-one-79vgf   1/1     Running   0          112s
```

Check labels of node

```
kubectl get nodes --show-labels
```

We should get output like:

```
NAME       STATUS   ROLES    AGE     VERSION   LABELS
minikube   Ready    master   8m43s   v1.17.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=minikube,kubernetes.io/os=linux,minikube.k8s.io/commit=cbda04cf6bbe65e987ae52bb393c10099ab62014,minikube.k8s.io/name=minikube,minikube.k8s.io/updated_at=2020_03_17T06_41_37_0700,minikube.k8s.io/version=v1.8.1,node-role.kubernetes.io/master=
```

## Specific Node

Edit `ds.yaml`, add the `nodeSelector` block.

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-one
spec:
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
        image: nginx
        ports:
        - containerPort: 80
      nodeSelector:
        test: "true"
```

Update the daemonset.

```
kubectl apply -f ds.yaml
```

Check Deamon.

```
kubectl get daemonset
```

We should get output like below, there's no replica running. Because the minikube node doesn't have label `test`.

```
NAME     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
ds-one   0         0         0       0            0           test=true       8m15s
```

Check number of pod.

```
kubectl get pods
```

We should get output like below, there's no pod running. Because the minikube node doesn't have label `test`.

```
No resources found in default namespace.
```

Add label to node.

```
kubectl label node minikube test=true
```

Check Deamon.

```
kubectl get daemonset
```

We should get output like:

```
NAME     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
ds-one   1         1         1       1            1           test=true       10m
```

Check number of pod.

```
kubectl get pods
```

We should get output like:

```
NAME           READY   STATUS    RESTARTS   AGE
ds-one-8dfzt   1/1     Running   0          41s
```

## Clean

```
kubectl label node minikube test-
kubectl delete -f ds.yaml
```
