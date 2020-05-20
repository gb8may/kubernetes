![helm](https://user-images.githubusercontent.com/35708820/82477925-8384f980-9aa6-11ea-91ca-9feb0d57b1fd.jpg)

### Install Helm

```
wget https://get.helm.sh/helm-v2.14.1-linux-amd64.tar.gz
tar zxf helm*gz
sudo cp linux-amd64/helm /usr/local/bin/
rm -rf helm* linux-amd64
kubectl -n kube-system create serviceaccount tiller
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account tiller
```
