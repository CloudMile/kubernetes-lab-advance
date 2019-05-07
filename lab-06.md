# Lab 06 - DNS configuration

Edit `pod.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - image: busybox:1.28
    name: busy
    command:
    - sleep
    - "3600"
```

```
kubectl apply -f pod.yaml
```

Check the dns policy.

```
kubectl get pod busybox -o yaml | grep dnsPolicy:
```

Check the `/etc/resolv.conf`

```
kubectl exec -it busybox -- cat /etc/resolv.conf
```

## Create custom dns server

Edit `custom-dns.yaml`

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: custom-dns
data:
  chosts: |
    4.3.2.1 www.acme.local
```

```
kubectl apply -f custom-dns.yaml
```

Edit `cdns.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: cdns
spec:
  containers:
  - image: k8s.gcr.io/dnsmasq:1.1
    name: cdns
    command:
    - dnsmasq
    - -k
    - --hostsdir
    - /hosts
    volumeMounts:
    - mountPath: /hosts
      name: hosts
  volumes:
  - name: hosts
    configMap:
      name: custom-dns
```

```
kubectl apply -f cdns.yaml
```

Get Pod private IP

```
kubectl get po -o wide
```

## Add custom dns record

Edit `kube-dns.yaml`, change `CUSTOM_DNS_POD_IP`

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: kube-dns
  namespace: kube-system
data:
  stubDomains: |
    {"acme.local": ["CUSTOM_DNS_POD_IP"]}
  upstreamNameservers: |
    ["8.8.8.8", "8.8.4.4"]
```

```
kubectl apply -f kube-dns.yaml
```

Test

```
kubectl run -it dnsutils --image k8s.gcr.io/dnsutils --restart=Never --rm sh

dig www.acme.local
```

## Clean

```
kubectl delete -f cdns.yaml
kubectl delete -f custom-dns.yaml
kubectl delete -f kube-dns.yaml
```
