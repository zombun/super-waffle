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

## Mariadb

install
```
helm install mariadb bitnami/mariadb -f ./apps/mariadb/values.yaml --namespace mariadb
```

use
```
 echo Primary: mariadb.mariadb.svc.cluster.local:3306

Administrator credentials:

  Username: root
  Password : $(kubectl get secret --namespace mariadb mariadb -o jsonpath="{.data.mariadb-root-password}" | base64 -d)

To connect to your database:

  1. Run a pod that you can use as a client:

      kubectl run mariadb-client --rm --tty -i --restart='Never' --image  docker.io/bitnami/mariadb:11.4.5-debian-12-r9 --namespace mariadb --command -- bash

  2. To connect to primary service (read/write):

      mysql -h mariadb.mariadb.svc.cluster.local -uroot -p ls25       

To upgrade this helm chart:

  1. Obtain the password as described on the 'Administrator credentials' section and set the 'auth.rootPassword' parameter as shown below:

      ROOT_PASSWORD=$(kubectl get secret --namespace mariadb mariadb -o jsonpath="{.data.mariadb-root-password}" | base64 -d)
      helm upgrade --namespace mariadb mariadb oci://registry-1.docker.io/bitnamicharts/mariadb --set auth.rootPassword=$ROOT_PASSWORD
```

## Wordpress

install

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
kubectl create namespace wordpress

helm install my-wordpress bitnami/wordpress -f ./apps/wordpress/values.yaml --namespace wordpress
```

```
my-wordpress.wordpress.svc.cluster.local (port 80)

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL and associate WordPress hostname to your cluster external IP:

   export CLUSTER_IP=$(minikube ip) # On Minikube. Use: `kubectl cluster-info` on others K8s clusters
   echo "WordPress URL: https://wordpress.beg.xx.zombun.org/"
   echo "$CLUSTER_IP  wordpress.beg.xx.zombun.org" | sudo tee -a /etc/hosts

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: admin
  echo Password: $(kubectl get secret --namespace wordpress my-wordpress -o jsonpath="{.data.wordpress-password}" | base64 -d)
```

## Metrics server


```
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm repo update
helm install metrics-server metrics-server/metrics-server -n kube-system -f ./apps/metrics-server/values.yaml
```