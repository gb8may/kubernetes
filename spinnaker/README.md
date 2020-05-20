![spinnaker](https://user-images.githubusercontent.com/35708820/82478149-dd85bf00-9aa6-11ea-9382-43f8b0c1ca57.png)


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

