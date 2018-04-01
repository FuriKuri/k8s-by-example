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

