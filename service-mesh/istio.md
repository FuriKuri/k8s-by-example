# Istio

## Prerequisite

See [https://istio.io/docs/setup/kubernetes/quick-start.html](https://istio.io/docs/setup/kubernetes/quick-start.html)

### kops

If is important that you have a cluster in version 1.9 or higher.

Add the following part to you cluster configuration:

```
$ kops edit cluster <cluster-name>

kubeAPIServer:
    admissionControl:
    - NamespaceLifecycle
    - LimitRanger
    - ServiceAccount
    - PersistentVolumeLabel
    - DefaultStorageClass
    - DefaultTolerationSeconds
    - MutatingAdmissionWebhook
    - ValidatingAdmissionWebhook
    - ResourceQuota
    - NodeRestriction
    - Priority
```

Do not forget to update your cluster:

```
$ kops update cluster --yes
$ kops rolling-update cluster --yes
```

The api-server should now have the admission control parameters:

```
$ for i in `kubectl get pods -nkube-system | grep api | awk '{print $1}'` ; do  kubectl describe pods -nkube-system $i | grep "/usr/local/bin/kube-apiserver"  ; done

      mkfifo /tmp/pipe; (tee -a /var/log/kube-apiserver.log < /tmp/pipe & ) ; exec /usr/local/bin/kube-apiserver --address=127.0.0.1 --admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,DefaultTolerationSeconds,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota,NodeRestriction,Priority ...
```

## Install

This will download and extract the latest in your current directory:

```
$ curl -L https://git.io/getLatestIstio | sh -
$ cd istio-0.6
$ export PATH=$PWD/bin:$PATH
```

Install in k8s:
```
$ kubectl apply -f install/kubernetes/istio.yaml
```

We are also install the mutating webhook admission controller. 

```
$ ./install/kubernetes/webhook-create-signed-cert.sh \
    --service istio-sidecar-injector \
    --namespace istio-system \
    --secret sidecar-injector-certs

$ kubectl apply -f install/kubernetes/istio-sidecar-injector-configmap-release.yaml

$ cat install/kubernetes/istio-sidecar-injector.yaml | \
     ./install/kubernetes/webhook-patch-ca-bundle.sh > \
     install/kubernetes/istio-sidecar-injector-with-ca-bundle.yaml

$ kubectl apply -f install/kubernetes/istio-sidecar-injector-with-ca-bundle.yaml
```

_With version 0.6 I had problems with the webhook-create-signed-cert.sh script. I removed the part in the script and saved theyaml output in the temporary file. The problem with the yaml content is that the api version and kind is missing and the secret name seems to be incorrect. So I've added 'apiVersion: v1' and 'kind: Secret' and renamed the secret to name: sidecar-injector-certs._

Now the istio namespace should have the following pods:

```
$ kubectl get pods -n istio-system
NAME                                      READY     STATUS        RESTARTS   AGE
istio-ca-59f6dcb7d9-fggjf                 1/1       Running       0          18m
istio-ingress-779649ff5b-qkgvf            1/1       Running       0          18m
istio-mixer-7f4fd7dff-j2lmw               3/3       Running       0          18m
istio-pilot-5f5f76ddc8-knmz5              2/2       Running       0          18m
istio-sidecar-injector-54578c9669-xz5gv   1/1       Running       0          25s
```

The Istio-Sidecar-injector will automatically inject Envoy containers into your application pods assuming running in namespaces labeled with istio-injection=enabled:

```
$ kubectl create namespace istio-demo
$ kubectl label namespace istio-demo istio-injection=enabled
```

## Tasks
