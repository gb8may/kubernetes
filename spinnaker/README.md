<img src="https://user-images.githubusercontent.com/35708820/82478149-dd85bf00-9aa6-11ea-9382-43f8b0c1ca57.png"  width="150" height="150">


## Install Spinnaker on Kubernetes

> *For official guide and documentation you can go to the [Spinnaker.io](https://www.spinnaker.io/setup/) website.*

#### 1. Installing Halyard
Halyard is a command-line administration tool that manages the lifecycle of your Spinnaker deployment, including writing & validating your deployment’s configuration, deploying each of Spinnaker’s microservices, and updating the deployment.
> *Halyard is compatible only with Debian/Linux and MacOS

##### Halyard with [Docker](https://docs.docker.com/engine/install/) container
> *Install Docker on host you use to access your Kubernetes Cluster (In my case, I'm using a Cloud9 Environment and EKS for cluster)

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
#### 2. Choose a Provider

- Set the provider. More information about providers [here](https://www.spinnaker.io/setup/install/providers/).
```
$ hal config provider kubernetes enable
```

#### 3. Choose an Environment

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
> *If you have a dynamic volume remove the "--persistence.enabled=false"*

#### 4. Choose a Storage Service

- Let's disable S3 versioning on minio and change storage type to minio/s3
> *Get into container again to run this commands
```
mkdir ~/.hal/default/profiles
echo "spinnaker.s3.versioning: false" > ~/.hal/default/profiles/front50-local.yml
hal config storage s3 edit --endpoint http://minio:9000 --access-key-id "myaccesskey" --secret-access-key "mysecretkey"
hal config storage s3 edit --path-style-access true
hal config storage edit --type s3
```

#### 5. Deploy Spinnaker

- Now let's deploy Spinnaker
```
$ hal deploy apply
```

- Let's change the service type to a load balancer
```
$ kubectl -n spinnaker edit svc spin-deck
$ kubectl -n spinnaker edit svc spin-gate
```

- Make sure that load balancer it's working and save the external address for both load balancers
```
$ kubectl get svc -n spinnaker
```

- Redeploy with addresses of load balancers
```
$ hal config security ui edit --override-base-url "http://<LoadBalancerIP>:9000"
$ hal config security api edit --override-base-url "http://<LoadBalancerIP>:8084"
$ hal deploy apply
```

Now, go to your browser and access "http://LoadBalancerIP:9000" and have fun!

![spinnaker-hello](https://user-images.githubusercontent.com/35708820/82512524-c36bd100-9ae6-11ea-83be-46a4a5cfad03.png)
