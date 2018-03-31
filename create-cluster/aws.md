# AWS

## kops

At the moment the easiest way to create a kubernetes cluster on AWS si to use the tools [kops](https://github.com/kubernetes/kops).

### Install

The current release can be found here: [https://github.com/kubernetes/kops/releases](https://github.com/kubernetes/kops/releases).

**Mac**

```
$ brew update && brew install kops
```

**Linux**

```
$ wget https://github.com/kubernetes/kops/releases/download/1.8.1/kops-linux-amd64
$ chmod +x kops-linux-amd64
$ mv kops-linux-amd64 /usr/local/bin/kops
```

### Setup

kops needs a S3 bucket to store the clusters configuration and a hosted zone for the running cluster.

```
$ aws s3api create-bucket --bucket kops-state-store-<account-id> --region eu-central-1 --create-bucket-configuration LocationConstraint=eu-central-1

$ aws route53 create-hosted-zone --name <your-domain> --caller-reference 1
```

### Create cluster

```
$ kops create cluster --zones=eu-central-1a <cluster-name>.<your-domain>
```

For example for my cluster

```
$ kops create cluster --zones=eu-central-1a --state s3://kops-state-store-111111111 kube.furikuri.net
```