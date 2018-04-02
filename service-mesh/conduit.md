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

