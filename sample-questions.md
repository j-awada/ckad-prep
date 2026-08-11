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
>ClusterIP services are only reachable from within the cluster since the IP is internal to the cluster. Here are some service types:
>* ClusterIP: default, exposes the service on a cluster internal IP
>* NodePort: exposes the service on each node's IP at a static port between 30000 to 32767
>* LoadBalancer: exposes the service externally using an external load balancer

Port forward the service to localhost:

```bash
k port-forward svc/myservice 8080:80
```
