# Kubernetes (k3s) setup and reverse https (LE) proxy

Setting up a server, with kubernetes and automatic ingress https let's encrypt

## MASTER NODE

  ### Tested on:

    |OS|Provider|Working|
    |-|-|-|
    |Debian 9|OVH|YES|
    |Debian 10|OVH|YES|
    |Debian 11|OVH|YES|

  ### Steps:

    * Edit `./cert-manager/kustomization.yaml`:
       * Replace `{{EMAIL}}` by your own **valid email**

    * *K3s* Install
       * ```
         curl -sfL https://get.k3s.io |  sh -
         ```

    * *Cert-manager* install
       * ```bash
         kubectl create namespace cert-manager
         kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.1.1/cert-manager.yaml 
         #./cert-manager/cert-manager.v1.1.1.yaml
         sleep 120
         ```

    * Create ClusterIssuer
       * ```
         kubectl apply -k ./cert-manager/
         ```



  ### Full set of commands

    ```
    git clone https://github.com/NewTechStack/Kubernetes_setup
    nano ./cert-manager/kustomization.yaml
    curl -sfL https://get.k3s.io |  sh -
    kubectl create namespace cert-manager
    kubectl apply -f ./cert-manager/cert-manager.v1.1.1.yaml
    sleep 120
    kubectl apply -k ./cert-manager/
    ```


  ### Link with an app

  Config your app inside a namespace `{{APP_NAMESPACE}}` (your ingress **must** be in the same namespace as your app), then edit `./examples/ingress-example.yaml` and change `{{NAME}}`, `{{DOMAIN}}`, `{{SERVICE}}` and `{{PORT}}` then run

  ```
  kubectl -n {{APP_NAMESPACE}} apply -f ./examples/ingress-example.yaml
  ```

