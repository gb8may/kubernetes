### Install Helm

```
wget https://get.helm.sh/helm-v3.2.0-linux-amd64.tar.gz
tar zxf helm-v3.2.0-linux-amd64.tar.gz
sudo cp linux-amd64/helm /usr/local/bin/
rm -rf helm* linux-amd64
kubectl -n kube-system create serviceaccount tiller
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account tiller
```
