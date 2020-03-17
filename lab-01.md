# Lab 01 - Using Volume

We use [KataCota](https://www.katacoda.com/) as playground.

This lab is on the environment: [Deploy Containers Using YAML](https://www.katacoda.com/courses/kubernetes/creating-kubernetes-yaml-definitions).

Click "START SCENARIO".

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

## Shared volume

Edit `pod-shared-volume.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - image: busybox
    name: busy
    command:
    - sleep
    - "3600"
    volumeMounts:
    - mountPath: /busy
      name: test
  - image: busybox
    name: box
    command:
    - sleep
    - "3600"
    volumeMounts:
    - mountPath: /busy
      name: test
  volumes:
  - name: test
    emptyDir: {}
```

Deploy Pod

```
kubectl apply -f pod-shared-volume.yaml
```

Add file in `busy` container in Pod.

```
kubectl exec -it busybox -c busy -- touch /busy/foo
```

Check file in `box` container in Pod.

```
kubectl exec -it busybox -c box -- ls /busy
```

Delete Pod.

```
kubectl delete -f pod-shared-volume.yaml
```

## Use configmaps as volume

Create configmap

```
kubectl create configmap dummy-config --from-literal=foo=bar
```

Edit `pod-configmap.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox-configmap
  namespace: default
spec:
  containers:
  - image: busybox
    name: busy
    command:
    - sleep
    - "3600"
    volumeMounts:
    - mountPath: /busy
      name: test
  volumes:
  - name: test
    configMap:
      name: dummy-config
```

Deploy Pod

```
kubectl apply -f pod-configmap.yaml
```

Check file.

```
kubectl exec -it busybox-configmap -- ls /busy
```

Delete Pod.

```
kubectl delete -f pod-configmap.yaml
```

## Use secret as volume

Create Secret

```
kubectl create secret generic dummy-secret --from-literal=foo=bar
```

Edit `pod-secret.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox-secret
  namespace: default
spec:
  containers:
  - image: busybox
    name: busy
    command:
    - sleep
    - "3600"
    volumeMounts:
    - mountPath: /busy
      name: test
  volumes:
  - name: test
    secret:
      secretName: dummy-secret
```

Deploy Pod

```
kubectl apply -f pod-secret.yaml
```

Check file.

```
kubectl exec -it busybox-secret -- ls /busy
```

Delete Pod.

```
kubectl delete -f pod-secret.yaml
```

## Use hostPath

Edit `pod-hostPath.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox-hostpath
  namespace: default
spec:
  containers:
  - image: busybox
    name: busy
    command:
    - sleep
    - "3600"
    volumeMounts:
    - mountPath: /busy
      name: test
  volumes:
  - name: test
    hostPath:
      path: "/data/busybox-test"
```

Deploy Pod

```
kubectl apply -f pod-hostPath.yaml
```

Add file.

```
kubectl exec -it busybox-hostpath -c busy -- touch /busy/foo
```

Check file.

```
minikube ssh -- ls /data/busybox-test
```

Delete Pod.

```
kubectl delete -f pod-hostpath.yaml
```

## Use Downward API

Edit `pod-downward.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox-downward
  labels:
    foo: bar
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c"]
      args:
      - while true; do
          if [[ -e /etc/podinfo/labels ]]; then
            echo -en '\n\n'; cat /etc/podinfo/labels; fi;
          sleep 5;
        done;
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
          readOnly: false
  volumes:
    - name: podinfo
      downwardAPI:
        items:
          - path: "labels"
            fieldRef:
              fieldPath: metadata.labels
```

Deploy Pod

```
kubectl apply -f pod-downward.yaml
```

Check logs.

```
kubectl logs busybox-downward
```

Delete Pod.

```
kubectl delete -f pod-downward.yaml
```

## Use persistent volume

### Create persistent volumes

Edit `pv-1g.yaml`

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-1g
  labels:
    type: local
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: "/data/pv-1g"
```

```
kubectl apply -f pv-1g.yaml
```

Edit `pv-3g.yaml`

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-3g
  labels:
    type: local
spec:
  capacity:
    storage: 3Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: "/data/pv-3g"
```

```
kubectl apply -f pv-3g.yaml
```

Check persistent volumes status.

```
kubectl get pv
```

Edit `pvc-2g.yaml`

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-2g
spec:
  storageClassName: ""
  accessModes:
  - ReadWriteOnce
  selector:
    matchLabels:
      type: local
  resources:
    requests:
      storage: 2Gi
```

```
kubectl apply -f pvc-2g.yaml
```

Check persistent volumes status.

```
kubectl get pvc,pv
```

Edit `pod-pvc.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pvc
  namespace: default
spec:
  containers:
  - image: busybox
    name: busy
    command:
    - sleep
    - "3600"
    volumeMounts:
    - mountPath: /busy
      name: test
  volumes:
  - name: test
    persistentVolumeClaim:
      claimName: pvc-2g
```

```
kubectl apply -f pod-pvc.yaml
```

Check persistent volumes status.

```
kubectl get pvc,pv
```

Add file.

```
kubectl exec -it busybox-pvc -c busy -- touch /busy/foo
```

Redeploy Pod

```
kubectl delete -f pod-pvc.yaml
kubectl apply -f pod-pvc.yaml
```

Check file

```
kubectl exec -it busybox-pvc -c busy -- ls /busy
```

## Clean

```
kubectl delete configmaps dummy-config
kubectl delete secret dummy-secret
kubectl delete -f pod-pvc.yaml
kubectl patch pvc pvc-2g -p '{"metadata":{"finalizers":null}}' # avoid delete task stuck
kubectl delete -f pvc-2g.yaml
kubectl delete -f pv-1g.yaml
kubectl delete -f pv-3g.yaml
```
