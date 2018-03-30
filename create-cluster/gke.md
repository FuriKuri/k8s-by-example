# GKE

One easy to way create a cluster is to use GKE from the Google Cloud Platform.

## Prerequisite

### gcloud CLI

**Mac** and **Linux**

```
$ curl https://sdk.cloud.google.com | bash

$ exec -l $SHELL

$ gcloud init
```

**Windows**

See [https://cloud.google.com/sdk/downloads\#windows](https://cloud.google.com/sdk/downloads#windows)

### kubectl CLI

Since we have gcloud installed, we can use the following command:

```
$ gcloud components install kubectl
```

## Create Cluster

```
$ gcloud container clusters create <cluster-name>
```

For example for project cluster-name furikuri

```
$ gcloud container clusters create furikuri

Creating cluster furikuri...done.
Created [https://container.googleapis.com/v1/projects/furi-111111/zones/us-central1-a/clusters/furikuri].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-central1-a/furikuri?project=furi-111111
kubeconfig entry generated for furikuri.
NAME      LOCATION       MASTER_VERSION  MASTER_IP     MACHINE_TYPE   NODE_VERSION  NUM_NODES  STATUS
furikuri  us-central1-a  1.8.8-gke.0     35.184.45.35  n1-standard-1  1.8.8-gke.0   3          RUNNING
```

## Access Cluster

```
$ kubectl get nodes
```

For example for fresh created cluster

```
$ kubectl get nodes

NAME                                      STATUS    ROLES     AGE       VERSION
gke-furikuri-default-pool-8bd501ae-7lk6   Ready     <none>    45s       v1.8.8-gke.0
gke-furikuri-default-pool-8bd501ae-l2pc   Ready     <none>    48s       v1.8.8-gke.0
gke-furikuri-default-pool-8bd501ae-wt2v   Ready     <none>    48s       v1.8.8-gke.0
```

## Delete Cluster

```
$ gcloud container clusters delete <cluster-name>
```



