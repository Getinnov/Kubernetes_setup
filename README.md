## Kubernetes (k3s) setup and reverse https (LE) proxy

Setting up a server, with kubernetes and automatic ingress https let's encrypt

### Tested on:

  |OS|HOST|Working|
  |-|-|-|
  |Debian 9|OVH|YES|

### Steps:

  * Edit `./cert-manager/kustomization.yaml`:
     * Replace `{{EMAIL}}` by your own **valid email**

  * *K3s* Install
     * ```
       curl -sfL https://get.k3s.io |  sh -
       ```

  * *Cert-manager* install
     * ```
       kubectl create namespace cert-manager
       kubectl apply -f ./cert-manager/cert-manager.v1.1.1.yaml
       #https://github.com/jetstack/cert-manager/releases/download/v1.1.1/cert-manager.crds.yaml
       sleep 120
       ```

  * Create ClusterIssuer
     * ```
       kubectl apply -k ./cert-manager/
       ```



### Full set of commands
  
  ```
  nano ./cert-manager/kustomization.yaml
  curl -sfL https://get.k3s.io |  sh -
  kubectl create namespace cert-manager
  kubectl apply -f ./cert-manager/cert-manager.v1.1.1.yaml
  sleep 120
  kubectl apply -k ./cert-manager/
  ```
  
  be sure to have the last version of cert-manager by using:
  ```
  URL=$(curl --silent "https://api.github.com/repos/jetstack/cert-manager/releases/latest" | jq -r '.assets[0].browser_download_url')
  kubectl apply -f $URL
  ```
  instead of
  ```
  kubectl apply -f ./cert-manager/cert-manager.v1.1.1.yaml
  ```

### Link with an app

Config your app inside a namespace `{{APP_NAMESPACE}}` (your ingress **must** be in the same namespace as your app), then edit `./examples/ingress-example.yaml` and change `{{NAME}}`, `{{DOMAIN}}`, `{{SERVICE}}` and `{{PORT}}` then run

```
kubectl -n {{APP_NAMESPACE}} apply -f ./examples/ingress-example.yaml
```

#### Update your cert-manager:
```
 URL=$(curl --silent "https://api.github.com/repos/jetstack/cert-manager/releases/latest" | jq -r '.assets[0].browser_download_url')
 kubectl apply -f $URL
```
