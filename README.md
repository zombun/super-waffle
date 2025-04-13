# super-waffle


apps - individual manifests
argocd-apps - deploy with argocd

## What is installed

- [Argo CD](https://argo-cd.readthedocs.io/en/stable/) for GitOps
- Kubernetes dashboard
- Traefik
- Metallb
- Ceph
- nginx sample app 

# Argo CD

## [install](https://argo-cd.readthedocs.io/en/stable/)

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## [dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

```
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath={.data.password} | base64 -d
kubectl port-forward svc/argocd-server -n argocd 8080:443 
```

# Kubernetes dashboard


## Install

```
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
kubectl apply -f apps/kubernetes-dashboard/auth.yaml 
```

## Create admin user token

```
kubectl -n kubernetes-dashboard create token admin-user 
```
## Forward

```
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443
```

For some reason the url that needs to be used is https://127.0.0.1:8443/ and https://localhost:8443 does not work (authentication fails)


# Ceph 

## install

```bash
kubectl apply -f argocd-apps/rook-ceph-app.yaml
```

[Check resource requirements](https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Kubernetes-Platform-v2_12:top-rook-ceph-resource-requirements-r.html)


## dashboard

username: admin

```
kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo
kubectl -n rook-ceph port-forward $(kubectl -n rook-ceph get pod -l app=rook-ceph-mgr -o jsonpath="{.items[0].metadata.name}") 7000:7000
```

## cli

TODO: https://rook.io/docs/rook/v1.14/Troubleshooting/ceph-toolbox/


## diagnostics

```
kubectl -n rook-ceph get service
```


### Metrics api

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl top nodes
```

