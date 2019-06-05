# Lab 05 - Create a GKE cluster

Qwiklabs: [Hello Node Kubernetes](https://www.qwiklabs.com/focuses/564?parent=catalog)

## Note 1:

For some commands, you will replcase `PROJECT_ID` with the prohect id.

```
docker build -t gcr.io/PROJECT_ID/hello-node:v1 .
```

In Cloud Shell, there're environment variables you can use, for example: `DEVSHELL_PROJECT_ID`.
You can use below command instand of above command.

```
docker build -t gcr.io/$DEVSHELL_PROJECT_ID/hello-node:v1 .
```

## Note 2:

In section __Roll out an upgrade to your service__, there's a typo.

The command

```
docker push gcr.io/PROJECT_ID/hello-node:v1
```

should be

```
docker push gcr.io/PROJECT_ID/hello-node:v2
```

replcase `PROJECT_ID` with the prohect id, or use environment variable `DEVSHELL_PROJECT_ID`.

----

Rrequirement

* gcloud
* kubectl

Document: https://cloud.google.com/sdk/install

Enable Cloud APIs

```
gcloud services enable container.googleapis.com
gcloud services enable cloudbuild.googleapis.com
gcloud services enable containerregistry.googleapis.com
gcloud services enable logging.googleapis.com
gcloud services enable monitoring.googleapis.com
```

```
gcloud beta container clusters create gke-zone-demo \
  --zone asia-east1-b \
  --enable-ip-alias \
  --enable-stackdriver-kubernetes
```

## Create a cluster

Single Zone Cluster

```
gcloud beta container clusters create gke-zone-demo \
  --zone asia-east1-b \
  --enable-ip-alias \
  --enable-stackdriver-kubernetes
```

Regional Cluster

```
gcloud beta container clusters create gke-region-demo \
  --region asia-east1 \
  --enable-ip-alias \
  --enable-stackdriver-kubernetes
```

## Access cluster and browse resources

Setup `kubectl` credentials

```
gcloud container clusters get-credentials gke-zone-demo \
  --zone asia-east1-b
```

Show `kubectl`'s current context

```
kubectl config current-context
```

Show cluster infomation

```
kubectl cluster-info
```
