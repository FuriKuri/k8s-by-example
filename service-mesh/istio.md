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

### Deploy demo app

```
$ kubectl create -n istio-demo -f https://raw.githubusercontent.com/FuriKuri/demo/master/deploy.yaml
```

### Egress traffic

```
cat <<EOF | istioctl create -f -
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  namespace: istio-demo
  name: httpbin-timeout-rule
spec:
  destination:
    service: httpbin.org
  http_req_timeout:
    simple_timeout:
      timeout: 3s
EOF
```

```
$ http http://127.0.0.1:8001/api/v1/namespaces/istio-demo/services/http:client:80/proxy/http/httpbin.org/delay/0
HTTP/1.1 200 OK
Content-Length: 846
Content-Type: text/plain; charset=utf-8
Date: Sat, 31 Mar 2018 17:01:35 GMT
Server: envoy
X-Envoy-Decorator-Operation: default-route
X-Envoy-Upstream-Service-Time: 187

{
    "args": {},
    "data": "",
    "files": {},
    "form": {},
    "headers": {
        "Accept-Encoding": "gzip",
        "Connection": "close",
        "Host": "httpbin.org",
        "User-Agent": "Go-http-client/1.1",
        "X-B3-Sampled": "1",
        "X-B3-Spanid": "e6b2268a1a529fe4",
        "X-B3-Traceid": "e6b2268a1a529fe4",
        "X-Envoy-Decorator-Operation": "httpbin-timeout-rule",
        "X-Envoy-Expected-Rq-Timeout-Ms": "3000",
        "X-Istio-Attributes": "CkoKCnNvdXJjZS51aWQSPBI6a3ViZXJuZXRlczovL2NsaWVudC1kZXBsb3ltZW50LTU0NDc4YzhiOWMtOXZ2aDguaXN0aW8tZGVtbwpDCg1zb3VyY2UubGFiZWxzEjJKMAoNCgNhcHASBmNsaWVudAofChFwb2QtdGVtcGxhdGUtaGFzaBIKMTAwMzQ3NDY1NwofCglzb3VyY2UuaXASEjIQAAAAAAAAAAAAAP//ZGACBw==",
        "X-Ot-Span-Context": "e6b2268a1a529fe4;e6b2268a1a529fe4;0000000000000000"
    },
    "origin": "35.157.82.188",
    "url": "http://httpbin.org/delay/0"
}

$ http http://127.0.0.1:8001/api/v1/namespaces/istio-demo/services/http:client:80/proxy/http/httpbin.org/delay/4
HTTP/1.1 200 OK
Content-Length: 24
Content-Type: text/plain; charset=utf-8
Date: Sat, 31 Mar 2018 17:01:40 GMT
Server: envoy
X-Envoy-Decorator-Operation: default-route
X-Envoy-Upstream-Service-Time: 3003

upstream request timeout

```