# Lab 05 - Create a GKE cluster

In this lab, we use [Qwiklabs](https://www.qwiklabs.com/).

Qwiklabs Lab: [Hello Node Kubernetes](https://www.qwiklabs.com/focuses/564?parent=catalog)

We will:
- Build a container image to Google Container Registry
- Create a Google Kubernetes Engine Cluster.
- Deploy a stateless application
- Expose the application to public internet
- Scale application replicas
- Update container image of application

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

How to push conatiner image to Google Container Registry.

Set the credentials for docker.

```
gcloud auth configure-docker
```

Push the specify format tag.

```
docker push gcr.io/$PROJECT_ID/hello-node:v1
```

----

Self Environment Rrequirement

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

## Access cluster and browse resources

Setup `kubectl` credentials

```
gcloud container clusters get-credentials gke-zone-demo \
  --zone asia-east1-b
```
