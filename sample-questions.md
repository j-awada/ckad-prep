## Sample questions

### Docker

* Create a Docker image _myimage_ using the Dockerfile in this directory and tag it with `1.0.1`
* Run a container from the image, the name of the container should be _mycontainer_
* Push the image to the registry _myregistry_ under the username _me_ with tag latest
* Save the OCI generated image in tar format under the name _myimagetar_ and tag v2

```bash
docker build -t myimage:1.0.1 .
docker run -d --name mycontainer myimage:1.0.1
docker tag myimage:1.0.1 myregistry/me:latest
docker push myregistry/me:latest
docker save myimage:1.0.1 > myimagetar.v2.tar
```

### Pod

* Create a pod named _mypod_ with image `nginx:latest`
* The pod should be exposed on port 80
* The pod should have an environment variable `KEY1=VALUE1`
* The pod should be tagged as `app=backend`
* Create a service that exposes the pod on port 80, name the service _myservice_

```bash
k run mypod --image=nginx:latest --port=80 --env=KEY1=VALUE1
k label po mypod app=backend
k expose po mypod --name=myservice --port=80
```

> **_NOTE_**
>
> ClusterIP services are only reachable from within the cluster since the IP is internal to the cluster. Here are some service types:
> * ClusterIP: default, exposes the service on a cluster internal IP
> * NodePort: exposes the service on each node's IP at a static port between 30000 to 32767
> * LoadBalancer: exposes the service externally using an external load balancer

Port forward the service to localhost:

```bash
k port-forward svc/myservice 8080:80
```

### CronJob

> **_NOTE_**
>
> \* \* \* \* \* --> day of week (0–7, both 0 and 7 = Sunday) | month (1–12) | day of month (1–31) | hour (0–23) | minute (0–59)
>
> \*: any value
>
> */N: every N units
>
> A-B: range (e.g. 1-5)
>
> A,B,C: list (e.g. 1,3,5)


* Create a Cron job with a name _cron1_ and `busybox` image scheduled for every 2 minutes
* Keep 3 success and 2 failed history configurations
* Terminate the pod after 22 seconds
* Add a random command into the container of the job
* Run a job from this cron jobs

```bash
k create cronjob cron1 --image=busybox --schedule="*/2 * * * *" --dry-run=client -o yaml
```

Modify the YAML output by using a mix of `k create cronjob -h` and `k explain cronjob.spec.jobTemplate.spec.template.spec.containers.command` to drill down into the attributes that you need to add.

```YAML
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: cron1
spec:
  failedJobsHistoryLimit: 2
  successfulJobsHistoryLimit: 3
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: cron1
    spec:
      activeDeadlineSeconds: 22
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - image: busybox
            name: cron1
            resources: {}
            command:
              - /bin/sh
              - -c
              - date;echo Hello world!
          restartPolicy: OnFailure
  schedule: '*/2 * * * *'
```

```bash
k apply -f - <<EOF
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: cron1
spec:
  failedJobsHistoryLimit: 2
  successfulJobsHistoryLimit: 3
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: cron1
    spec:
      activeDeadlineSeconds: 22
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - image: busybox
            name: cron1
            resources: {}
            command:
              - /bin/sh
              - -c
              - date;echo Hello world!
          restartPolicy: OnFailure
  schedule: '*/2 * * * *'
EOF
```

```bash
# Run a job from the CronJob
k create job job1 --from=cronjob/cron1
```

> **_NOTE_**
>
> One CronJob object is like one line of a crontab (cron table) file on a Unix system.
>
> A CronJob creates and owns Job objects, each job is its own independent resource.
>
> A job's `ownerReferences` points back to the CronJob

### Events

* List all events of a pod called __r327dc2f9__

```bash
# Search for an attribute to use
k get events --help

# Look at the event JSON object
k get events -o json

# Use --field-selector
k get events --field-selector involvedObject.name=r327dc2f9
```

### Network policies and labels

* Grant access to an app without changing the network policy. The network policy allows ingress for apps matching this label `web-access: true`.

```bash
# Add a new label to the app
k label app web-access=true
```

### Secrets

* Creating a secret

```bash
# List secrets types
k create secret --help
#  docker-registry   Create a secret for use with a Docker registry
#  generic           Create a secret from a local file, directory, or literal value
#  tls               Create a TLS secret

k create secret generic test-secret --dry-run=client -o yaml

# Create a secret with key/value pair
k create secret generic --help
k create secret generic test-secret --namespace default --from-literal key1=value1 --from-literal key2=value2 --dry-run=client -o yaml
```

* Attaching a secret to a pod

```bash
# Get deployment help
k create deploy --help

k create deployment test-deploy --image=nginx:latest --dry-run=client -o yaml
k explain deployment.spec.template.spec.containers.env.valueFrom
```

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: test-deploy
  name: test-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test-deploy
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test-deploy
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        env:
        - name: key1
          valuesFrom:
            secretKeyRef:
              name: test-secret
              key: key1
        - name: key2
          valuesFrom:
            secretKeyRef:
              name: test-secret
              key: key2
```
