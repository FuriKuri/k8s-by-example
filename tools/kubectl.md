# kubectl

kubectl is the command line tool for your Kubernetes cluster.

## Install

**Mac**

```
brew install kubectl
```

**Linux**

```
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
$ chmod +x ./kubectl
$ sudo mv ./kubectl /usr/local/bin/kubectl
```

**Windows**

```
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.10.0/bin/windows/amd64/kubectl.exe
```

If **gcloud** is installed

```
$ gcloud components install kubectl
```