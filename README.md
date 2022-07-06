# Kubernetes (k3s) setup and reverse https (LE) proxy

  * [Setting up a master node, with K3S, automatic traefik ingress https using let's encrypt](#master-node)
  * [Adding node to your cluster](#agent-node)
  * [Changing the maximum number on pods per node](#edit-pod-limit)
  * Monitoring this cluster

### Tested on:

  | OS        | Provider | Master Node | Agent Node |
  | :-------: | :------: | :---------: | :--------: |
  | Debian 9  | OVH      | ✅          | ✅        |
  | Debian 10 | OVH      | ✅          | ✅        |
  | Debian 11 | OVH      | ✅          | ✅        |


## MASTER NODE:

### Installation and setup step by step:

These steps will allow you to install k3s-server on a new master node and have an automatic traefik ingress https using let's encrypt


  * Edit `./cert-manager/kustomization.yaml`:
     * ```bash
       EMAIL=cluster.acme@newteckstack.fr # Your email
       sed -i "s/{{EMAIL}}/$EMAIL/g" ./cert-manager/kustomization.yaml
       ```

  * *K3s* Install
     * ```
       curl -sfL https://get.k3s.io |  sh -
       ```

  * *Cert-manager* install
     * ```bash
       kubectl create namespace cert-manager
       kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.7.1/cert-manager.yaml
       sleep 120
       ```
      ℹ A copy of [cert-manager](https://github.com/jetstack/cert-manager) v1.7.1 is available inside `cert-manager/_sources/cert-manager.v1.7.1.yaml`([here](./cert-manager/_sources/cert-manager.v1.7.1.yaml))

  * Create ClusterIssuer
     * ```bash
       kubectl apply -k ./cert-manager/
       ```

### Installation script:
  
  ```bash
  EMAIL=cluster.acme@newteckstack.fr # Your email
  sed -i "s/{{EMAIL}}/$EMAIL/g" ./cert-manager/kustomization.yaml
  curl -sfL https://get.k3s.io |  sh -
  kubectl create namespace cert-manager
  kubectl apply -f ./cert-manager/_sources/cert-manager.v1.7.1.yaml
  sleep 120
  kubectl apply -k ./cert-manager/
  ```
  
  Common errors:
  - `Error from server (InternalError): error when creating "./": Internal error occurred: failed calling webhook "webhook.cert-manager.io": failed to call webhook: Post "https://cert-manager-webhook.cert-manager.svc:443/mutate?timeout=10s": x509: certificate signed by unknown authority`
    ```
    kubectl delete mutatingwebhookconfiguration.admissionregistration.k8s.io cert-manager-webhook
    kubectl delete validatingwebhookconfigurations.admissionregistration.k8s.io cert-manager-webhook
    ```
  

### Link with an app

* Config your app inside a namespace `{{APP_NAMESPACE}}`<br>
  ⚠️ Your ingress **must** be in the same namespace as your app
  
* Edit `./cert-manager/_examples/ingress-example.yaml` and change:
  * `{{NAME}}`
  * `{{DOMAIN}}`
  * `{{SERVICE}}`
  * `{{PORT}}` 
  
* Then run
  ```
  APP_NAMESPACE=test # your app namespace
  kubectl -n $APP_NAMESPACE apply -f ./cert-manager/_examples/ingress-example.yaml
  ```


## AGENT-NODE:


### Installation and setup step by step:

These steps will allow you to install k3s-agent on a new node and connect it with an existing cluster.

 * On your master node:
      * get your ip:
        ```
        kubectl get service kubernetes -o jsonpath='{.spec.clusterIP}'; echo
        ```
      * get your node token:
        ```bash
        cat /var/lib/rancher/k3s/server/node-token
        # example KXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX::server:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        ```
      * generate command:
        ```
        IP=$(kubectl get service kubernetes -o jsonpath='{.spec.clusterIP}'; echo)
        NODETOKEN=$(cat /var/lib/rancher/k3s/server/node-token)
        echo 'curl -sfL https://get.k3s.io | K3S_URL="https://'$IP':6443" K3S_TOKEN="'$NODETOKEN'" INSTALL_K3S_VERSION=v1.23.4+k3s1 sh -'
        ```
      
 * On your new node :
      * ```
        IP=XX.XX.XX.XX # Your master node ip
        NODETOKEN=KXXXXXXXXXXXXXXXXXXXXXX::server:XXXXXXXXXXXXXXXXXXXXXXXXX # your node token
        curl -sfL https://get.k3s.io | K3S_URL="https://$IP:6443" K3S_TOKEN="$NODETOKEN" INSTALL_K3S_VERSION=v1.23.4+k3s1 sh -
        ```
        or the generated command

## EDIT POD LIMIT

To update your existing installation with an increased max-pods, add a kubelet config file into a k3s, we will use `/etc/rancher/k3s/kubelet.config` :
 * Edit (or create) `/etc/rancher/k3s/kubelet.config`:
      * ```bash
        MAXPOD=250 # The number of pods that you want
        mkdir -p /etc/rancher/k3s
        cat > /etc/rancher/k3s/kubelet.config <<EOF
        apiVersion: kubelet.config.k8s.io/v1beta1
        kind: KubeletConfiguration
        maxPods: $MAXPOD
        EOF
        ```
      
 * On your master node edit `/etc/systemd/system/k3s.service` to change the k3s server args:
      * ```
        ExecStart=/usr/local/bin/k3s \
            server \
                '--kubelet-arg=config=/etc/rancher/k3s/kubelet.config'
        ```
        
      ⚠️ **If you have any line after `server \` you may want to keep them**
        
 * On your agent node edit `/etc/systemd/system/k3s-agent.service` to change the k3s server args:
      * ```
        ExecStart=/usr/local/bin/k3s \
            agent \
                '--kubelet-arg=config=/etc/rancher/k3s/kubelet.config'
        ```
        
      ⚠️ **If you have any line after `agent \` you may want to keep them** 
         
 * Reload systemctl to pick up the service change then restart k3s:
      * ```
        systemctl daemon-reload
        systemctl restart k3s
        ```
       if you encounter an `Failed to allocate` error `sysctl fs.inotify.max_user_instances=512`
