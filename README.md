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

## Set up kubernetes dashboard(optional)
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```