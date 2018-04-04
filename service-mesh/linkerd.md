# Linkerd

## Install

```
$ kubectl create ns linkerd
$ kubectl apply -f https://raw.githubusercontent.com/linkerd/linkerd-examples/master/k8s-daemonset/k8s/servicemesh.yml
```

## Deploy

Add the following part in the deployment file:

```
        env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: http_proxy
          value: $(NODE_NAME):4140
```

## Open dashboard

```
$ kubectl -n linkerd port-forward $(kubectl -n linkerd get pod -l app=l5d -o jsonpath='{.items[0].metadata.name}') 9990
```