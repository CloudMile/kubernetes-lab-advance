# Lab 06 - DNS configuration

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

## Deploy a Client Pod

Create a new file `pod.yaml`

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

Deploy Pod

```
kubectl apply -f pod.yaml
```

Check the dns policy.

```
kubectl get pod busybox -o yaml | grep dnsPolicy:
```

We should get output like:

```
  dnsPolicy: ClusterFirst
```

Check the `/etc/resolv.conf`

```
kubectl exec -it busybox -- cat /etc/resolv.conf
```

We should get output like:

```
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

## Create custom dns server

Create a new file `custom-dns.yaml`, we define a DNS A Record `www.acme.local` to `4.3.2.1`.

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: custom-dns
data:
  chosts: |
    4.3.2.1 www.acme.local
```

Deploy ConfigMap

```
kubectl apply -f custom-dns.yaml
```

Create a new file  `cdns.yaml`

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

Deplay the Custome DNS Pod

```
kubectl apply -f cdns.yaml
```

Get Pod private IP

```
kubectl get po -o wide
```

We should get output like:

```
NAME      READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
busybox   1/1     Running   0          18m   172.18.0.4   minikube   <none>           <none>
cdns      1/1     Running   0          10s   172.18.0.5   minikube   <none>           <none>
```

Copy the IP of `cdns` Pod for next section.

## Add custom dns record with CoreDNS

Get the corefile

```
kubectl get configmap coredns -n kube-system -o yaml > coredns.yaml
cp coredns.yaml coredns-bak.yaml
```

Edit `coredns.yaml` with below content, and change `CUSTOM_DNS_POD_IP` to the Custom DNS Pod IP.

```
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
    acme.local:53 {
      errors
      cache 30
      forward . CUSTOM_DNS_POD_IP
    }
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
```

Deploy ConfigMap

```
kubectl apply -f coredns.yaml
```

Force Reload ConfigMap

```
kubectl delete pods -n kube-system -l k8s-app=kube-dns
```

Wait for 30 secondes, and test.

```
kubectl run -it dnsutils --image k8s.gcr.io/dnsutils --restart=Never --rm sh

# dig www.acme.local
```

We should get output like:

```
; <<>> DiG 9.8.4-rpz2+rl005.12-P1 <<>> www.acme.local
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58458
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;www.acme.local.                        IN      A

;; ANSWER SECTION:
www.acme.local.         5       IN      A       4.3.2.1

;; Query time: 0 msec
;; SERVER: 10.96.0.10#53(10.96.0.10)
;; WHEN: Thu Mar 19 09:58:26 2020
;; MSG SIZE  rcvd: 62
```

If it doesn't work, force reload ConfigMap

```
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

Reference: https://coredns.io/2017/07/23/corefile-explained/

## Add custom dns record with kube-dns(Optional)

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

Test.

```
kubectl run -it dnsutils --image k8s.gcr.io/dnsutils --restart=Never --rm sh

# dig www.acme.local
```

We should get output like:

```
; <<>> DiG 9.8.4-rpz2+rl005.12-P1 <<>> www.acme.local
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58458
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;www.acme.local.                        IN      A

;; ANSWER SECTION:
www.acme.local.         5       IN      A       4.3.2.1

;; Query time: 0 msec
;; SERVER: 10.96.0.10#53(10.96.0.10)
;; WHEN: Thu Mar 19 09:58:26 2020
;; MSG SIZE  rcvd: 62
```

## Clean

```
kubectl delete -f pod.yaml
kubectl delete -f cdns.yaml
kubectl delete -f custom-dns.yaml

# if use coredns
kubectl apply -f coredns-bak.yaml

# if use kube-dns
kubectl delete -f kube-dns.yaml
```
