# Lab 04 - Using Job

Edit `job.yaml`

```
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  completions: 1
  parallelism: 1
  backoffLimit: 4
```

```
kubectl apply -f job.yaml
```

Check status.

```
kubectl get jobs
kubectl get pods
```

## Change completions

In other terminal

```
kubectl get po -w
```

Edit `job.yaml`

```
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  completions: 5
  parallelism: 1
  backoffLimit: 4
```

```
kubectl delete -f job.yaml
kubectl apply -f job.yaml
```

## Change parallelism

Edit `job.yaml`

```
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  completions: 10
  parallelism: 3
  backoffLimit: 4
```

```
kubectl delete -f job.yaml
kubectl apply -f job.yaml
```

Delete job

```
kubectl delete -f job.yaml
```

## CronJob

Edit `cronjob.yaml`

```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: reporting-cron-job
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      completions: 3
      parallelism: 3
      template:
        spec:
          containers:
            - name: busybox
              image: busybox
              command: ['expr', '3', '+', '2']
          restartPolicy: Never
```

```
kubectl apply -f cronjob.yaml
```

```
kubectl get pods -w
```

Wait 5 minutes.

```
kubectl delete -f cronjob.yaml
```
