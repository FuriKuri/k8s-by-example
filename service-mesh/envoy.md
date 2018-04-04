# Envoy / Ambassador

## Install

```
$ kubectl apply -f https://www.getambassador.io/yaml/ambassador/ambassador-rbac.yaml
```

```
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  labels:
    service: ambassador
  name: ambassador
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    service: ambassador
EOF
```

## Usage

```
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: google
  annotations:
    getambassador.io/config: |
      ---
      apiVersion: ambassador/v0
      kind:  Mapping
      name:  google_mapping
      prefix: /google/
      service: https://google.com:443
      host_rewrite: www.google.com
spec:
  type: ClusterIP
  clusterIP: None
EOF
```

```
$ LB_ADDRESS=$(kubectl get svc ambassador -o jsonpath="{.status.loadBalancer.ingress[0].*}")
$ http "http://$LB_ADDRESS/google/"
HTTP/1.1 302 Found
alt-svc: hq=":443"; ma=2592000; quic=51303432; quic=51303431; quic=51303339; quic=51303335,quic=":443"; ma=2592000; v="42,41,39,35"
cache-control: private
content-length: 267
content-type: text/html; charset=UTF-8
date: Wed, 04 Apr 2018 08:40:37 GMT
location: https://www.google.de/?gfe_rd=cr&dcr=0&ei=hY_EWuv4Fcr68Aeo8Yto
referrer-policy: no-referrer
server: envoy
x-envoy-upstream-service-time: 16

<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>302 Moved</TITLE></HEAD><BODY>
<H1>302 Moved</H1>
The document has moved
<A HREF="https://www.google.de/?gfe_rd=cr&amp;dcr=0&amp;ei=hY_EWuv4Fcr68Aeo8Yto">here</A>.
</BODY></HTML>
```