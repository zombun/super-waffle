# super-waffle


apps - individual manifests
argocd-apps - deploy with argocd

## What is installed

- [argocd](https://argo-cd.readthedocs.io/en/stable/) for GitOps
- Kubernetes dashboard
- Traefik
- Metallb
- Ceph
- nginx sample app 




## Argocd

```
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath={.data.password} | base64 -d
kubectl port-forward svc/argocd-server -n argocd 8080:443 
```


## Ceph dashboard

username: admin

```
kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo
kubectl -n rook-ceph port-forward $(kubectl -n rook-ceph get pod -l app=rook-ceph-mgr -o jsonpath="{.items[0].metadata.name}") 7000:7000
```