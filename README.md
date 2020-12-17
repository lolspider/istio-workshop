# istio-workshop

## Download vagrant and virtualbox
* (vagrant)[https://www.vagrantup.com/downloads.html]
* (virtualbox)[https://www.virtualbox.org/]

## Starting VM

This takes a while to download vm image.

```
vagrant up
```

## Connect your VM

```
vagrant ssh default
```

## Deploy CNI (flannel)

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

## [Optional] Set up kubernetes dashboard
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

## Be able to scheduler Pods on local node
```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

## Install Istio
```
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.8.1
echo "export PATH=$PWD/bin:$PATH" >> ~/.bashrc
```

## Add a namespace label to instruct Istio to automatically inject Envoy sidecar proxies when you deploy your application later
```
kubectl label namespace default istio-injection=enabled
```


## Deploy the sample application
```
cd istio-1.8.1
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

## Open the application to outside traffic

```
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml

export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')

export HOST=$(ip -4 address show dev eth1 | grep inet | awk '{print $2}' | cut -f1 -d/)

echo "http://$HOST:$INGRESS_PORT/productpage"
```

## View the dashboard

### Install Kiali and the other addons
```
kubectl apply -f samples/addons
```

### Access Kiali
```
istioctl dashboard kiali --address $HOST
```

## Request Routing

### Route all traffic for the reviews service to the version reviews:v1
```
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
```

### clean up
```
kubectl delete -f samples/bookinfo/networking/virtual-service-all-v1.yaml
```

### Route based on user identity
```
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
```

### clean up
```
kubectl delete -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
```


## Request Timeouts

```
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
kubectl apply -f istio/demo/request-timeout-p1.yaml
```
Then access the Bookinfo URL in you browser, there is a 2 second delay whenever you refresh the page.

```
kubectl apply -f istio/demo/request-timeout-p2.yaml
```

### clean up
```
kubectl delete -f samples/bookinfo/networking/virtual-service-all-v1.yaml
```

## Circuit Breaking

### Deploy httpbin 
```
kubectl apply -f samples/httpbin/httpbin.yaml
```

### Adding a client to test
```
kubectl apply -f samples/httpbin/sample-client/fortio-deploy.yaml
export FORTIO_POD=$(kubectl get pods -lapp=fortio -o 'jsonpath={.items[0].metadata.name}')
kubectl exec "$FORTIO_POD" -c fortio -- /usr/bin/fortio curl -quiet http://httpbin:8000/get
```

```
kubectl exec "$FORTIO_POD" -c fortio -- /usr/bin/fortio load -c 2 -qps 0 -n 20 -loglevel Warning http://httpbin:8000/get
kubectl exec "$FORTIO_POD" -c fortio -- /usr/bin/fortio load -c 3 -qps 0 -n 30 -loglevel Warning http://httpbin:8000/get

kubectl exec "$FORTIO_POD" -c istio-proxy -- pilot-agent request GET stats | grep httpbin | grep pending (optional)
```
### clean up
```
kubectl delete destinationrule httpbin
kubectl delete deploy httpbin fortio-deploy
kubectl delete svc httpbin fortio

```


