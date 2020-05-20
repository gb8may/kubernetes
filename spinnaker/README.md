![spinnaker](https://user-images.githubusercontent.com/35708820/82478149-dd85bf00-9aa6-11ea-9382-43f8b0c1ca57.png)
<img src="https://user-images.githubusercontent.com/35708820/82478149-dd85bf00-9aa6-11ea-9382-43f8b0c1ca57.png"  width="100" height="100">


## Install Spinnaker

> *For official guide and documentation you can go to the [Spinnaker.io](https://www.spinnaker.io/setup/) website.*

#### Installing Halyard

Creating a Halyard with [Docker](https://docs.docker.com/engine/install/) container
> *Install Docker on the main host you use to access your Kubernetes Cluster (In my case, I'm using EKS)

- Download and run Halyard Docker image
```
$ mkdir ~/.hal
$ docker run --name halyard -v ~/.hal:/home/spinnaker/.hal -v ~/.kube/config:/home/spinnaker/.kube/config -d gcr.io/spinnaker-marketplace/halyard:stable
```

- Get into container
```
$ docker exec -it halyard bash
```

- Inside the container, check if you can run Kubernetes commands
```
$ kubectl get nodes
```

- Set the provider. More information about providers [here](https://www.spinnaker.io/setup/install/providers/).
```
$ hal config provider kubernetes enable
```

- Add a Kubernetes account
```
$ hal config provider kubernetes account add my-k8s-account --provider-version v2 --context $(kubectl config current-context)
```

- Enable artifacts
```
$ hal config features edit --artifacts true
```

- Set whereb to install Spinnaker
```
$ hal config deploy edit --type distributed --account-name my-k8s-account
```

- Create a Spinnaker namespace and install minio on your Kubernetes cluster
> *Get out of the container, and run this commands on the container host.
```
$ kubectl create ns spinnaker

# For Helm v3+
$ helm install minio --namespace spinnaker --set accessKey="myaccesskey" --set secretKey="mysecretkey" --set persistence.enabled=false stable/minio

#For Helm v2+
$ helm install --name minio --namespace spinnaker --set accessKey="myaccesskey" --set secretKey="mysecretkey" --set persistence.enabled=false stable/minio
```

- Let's disable S3 versioning on minio and change storage type to minio/s3
> *Get into container again to run this commands
```
mkdir ~/.hal/default/profiles
echo "spinnaker.s3.versioning: false" > ~/.hal/default/profiles/front50-local.yml
hal config storage s3 edit --endpoint http://minio:9000 --access-key-id "myaccesskey" --secret-access-key "mysecretkey"
hal config storage s3 edit --path-style-access true
hal config storage edit --type s3
```

- Now let's deploy Spinnaker
```
$ hal deploy apply
```

- Now let's change the service type to a load balancer
```
$ kubectl -n spinnaker edit svc spin-deck
$ kubectl -n spinnaker edit svc spin-gate
```

- Make sure that load balancer it's working and save the external address for both load balancers
```
$ kubectl get svc -n spinnaker
```

- Now deploy again with addresses of load balancers
```
$ hal config security ui edit --override-base-url "http://<LoadBalancerIP>:9000"
$ hal config security api edit --override-base-url "http://<LoadBalancerIP>:8084"
$ hal deploy apply
```

Have fun!
