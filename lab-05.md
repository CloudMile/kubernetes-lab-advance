# Lab 05 - Create a GKE cluster

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
