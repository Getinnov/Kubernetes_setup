# Kubernetes_setup

Setting up a server, with kubernetes and automatic ingress https let's encrypt

* Tested on:

  |OS|HOST|Working|
  |-|-|-|
  |Debian 9|OVH|YES|

* Steps:

  1. Edit `./cert-manager/kustomization.yaml`:
     * Replace `{{EMAIL}}` by your own *valid* email

  2. *K3s* Install
     * ```
       curl -sfL https://get.k3s.io |  sh -
       ```

  3. *Cert-manager* install
     * ```
       kubectl create namespace cert-manager
       kubectl apply -f ./sources/cert-manager.v0.11.0.yaml
       sleep 120
       ```

  4. Create ClusterIssuer
     * ```
       kubectl apply -k ./cert-manager/
       ```



* Full set of commands
  ```
  curl -sfL https://get.k3s.io |  sh -
  kubectl create namespace cert-manager
  kubectl apply -f ./sources/cert-manager.v0.11.0.yaml
  sleep 120
  kubectl apply -k ./cert-manager/
  ```

Config your app inside a namespace `{{APP_NAMESPACE}}`, then edit `./sources/ingress.yaml` and change `{{HOST}}`, `{{PORT}}`, `{{NAME}}` and `{{SERVICE}}`

```
kubectl -n {{APP_NAMESPACE}} apply -f ./sources/ingress.yaml
```

/!\ Caution: your ingress have to be in the same namespace as your app

