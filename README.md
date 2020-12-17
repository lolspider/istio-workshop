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

export HOST=$(ip addr show eth1 | grep inet | grep -v inet6 | awk -F '[ /]' '{print $6}')

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

## Circuit Breaking