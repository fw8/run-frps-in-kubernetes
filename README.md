# Demo setup for a reverse tunnel to a local running service

The idea is to run a [frp](https://github.com/fatedier/) tunnel server in a kubernetes cluster in the cloud. Then you can establish connections to your local running applications using an **ingress** service running in the kubernetes cluster. Of course there are other use cases as well.

## Deploy the server

```sh
kubernetes apply -k server
```

## Start a local client

Find the IP of the kubernetes service to connect to from your **frp** client:

```bash
export FRP_SERVER_ADDR=`kubectl get -n frps svc/frps -o json | jq -r '.status.loadBalancer.ingress[0].ip'`
export FRP_SERVER_PORT=`kubectl get -n frps svc/frps -o json | jq '.spec.ports[]|select(.name=="frps").port'`
```

## Start the local service:

```bash
docker compose up -d
```

## Test the tunnel

Create a pod in the `frps` namespace and call the local running echo service through the tunnel:

```bash
kubectl run -it --rm=True --restart=Never -n frps --image curlimages/curl curlpod -- \
  curl -H "Host: www.test.de" http://http-tunnel-proxy:8080
```
