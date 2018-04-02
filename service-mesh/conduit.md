# Conduit

## Install

```
$ curl https://run.conduit.io/install | sh
$ export PATH=$PATH:$HOME/.conduit/bin
$ conduit install | kubectl apply -f -
```

## Deploy

```
$ curl https://raw.githubusercontent.com/FuriKuri/demo/master/deploy.yaml | conduit inject - | kubectl apply -f -
```

The command `conduit inject` will add an additional container and an additional init-container to your deployment:

```
      containers:
      - env:
        - name: CONDUIT_PROXY_LOG
          value: warn,conduit_proxy=info
        - name: CONDUIT_PROXY_CONTROL_URL
          value: tcp://proxy-api.conduit.svc.cluster.local:8086
        - name: CONDUIT_PROXY_CONTROL_LISTENER
          value: tcp://0.0.0.0:4190
        - name: CONDUIT_PROXY_PRIVATE_LISTENER
          value: tcp://127.0.0.1:4140
        - name: CONDUIT_PROXY_PUBLIC_LISTENER
          value: tcp://0.0.0.0:4143
        - name: CONDUIT_PROXY_NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: CONDUIT_PROXY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: CONDUIT_PROXY_POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        image: gcr.io/runconduit/proxy:v0.3.1
        imagePullPolicy: IfNotPresent
        name: conduit-proxy
        ports:
        - containerPort: 4143
          name: conduit-proxy
        resources: {}
        securityContext:
          runAsUser: 2102
      initContainers:
      - args:
        - --incoming-proxy-port
        - "4143"
        - --outgoing-proxy-port
        - "4140"
        - --proxy-uid
        - "2102"
        - --inbound-ports-to-ignore
        - "4190"
        image: gcr.io/runconduit/proxy-init:v0.3.1
        imagePullPolicy: IfNotPresent
        name: conduit-init
        resources: {}
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
          privileged: false
```

## Usage

### Example request

```
$ kubectl proxy
$ http http://127.0.0.1:8001/api/v1/namespaces/default/services/http:client:80/proxy/http/server/host
```

```
$ LB_ADDRESS=$(kubectl get svc client -o jsonpath="{.status.loadBalancer.ingress[0].*}")
$ http "http://$LB_ADDRESS/host"
```

### Dashboard

```
conduit dashboard
```

### CLI

```
$ conduit stat deployments
NAME                        REQUEST_RATE   SUCCESS_RATE   P50_LATENCY   P99_LATENCY
default/client-deployment         0.1rps        100.00%           0ms           0ms
```

```
$ conduit tap deploy default/client-deployment
req id=0:11 src=100.96.2.1:35447 dst=100.96.2.5:8080 :method=GET :authority=a89ab2a87364511e8950202637bdd010-2032770816.eu-central-1.elb.amazonaws.com :path=/host
rsp id=0:11 src=100.96.2.1:35447 dst=100.96.2.5:8080 :status=200 latency=274µs
end id=0:11 src=100.96.2.1:35447 dst=100.96.2.5:8080 duration=89µs response-length=34B
req id=0:12 src=100.96.2.1:35449 dst=100.96.2.5:8080 :method=GET :authority=a89ab2a87364511e8950202637bdd010-2032770816.eu-central-1.elb.amazonaws.com :path=/host
rsp id=0:12 src=100.96.2.1:35449 dst=100.96.2.5:8080 :status=200 latency=312µs
end id=0:12 src=100.96.2.1:35449 dst=100.96.2.5:8080 duration=84µs response-length=34B
```