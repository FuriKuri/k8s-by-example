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