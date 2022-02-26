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
       EMAIL= # Your email
       sed -i "s/{{EMAIL}}/$EMAIL/g" ./cert-manager/kustomization.yaml
       ```

  * *K3s* Install
     * ```
       curl -sfL https://get.k3s.io |  sh -
       ```

  * *Cert-manager* install
     * ```bash
       kubectl create namespace cert-manager
       kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.1.1/cert-manager.yaml 
       sleep 120
       ```
     * ℹ A copy of [cert-manager v1.1.1](https://github.com/jetstack/cert-manager/releases/download/v1.1.1/cert-manager.yaml) is available inside `cert-manager/_sources/cert-manager.v1.1.1.yaml`([here](./cert-manager/_sources/cert-manager.v1.1.1.yaml))

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
  kubectl apply -f ./cert-manager/cert-manager.v1.1.1.yaml
  sleep 120
  kubectl apply -k ./cert-manager/
  ```
  

### Link with an app

Config your app inside a namespace `{{APP_NAMESPACE}}` (your ingress **must** be in the same namespace as your app), then edit `./examples/ingress-example.yaml` and change `{{NAME}}`, `{{DOMAIN}}`, `{{SERVICE}}` and `{{PORT}}` then run

```
kubectl -n {{APP_NAMESPACE}} apply -f ./examples/ingress-example.yaml
```

## AGENT-NODE:


### Installation and setup step by step:

These steps will allow you to install k3s-agent on a new node and connect it with an existing cluster.

 * On your master node :
      * ```bash
        cat /var/lib/rancher/k3s/server/node-token
        # example KXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX::server:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        ```
      * This is your node token
      
 * On your new node :
      * ```
        IP=XX.XX.XX.XX # your master node ip
        NODETOKEN=KXXXXXXXXXXXXXXXXXXXXXX::server:XXXXXXXXXXXXXXXXXXXXXXXXX # your node token
        curl -sfL https://get.k3s.io | K3S_URL="https://$IP:6443" K3S_TOKEN="$NODETOKEN" sh -
        ```

## EDIT POD LIMIT

To update your existing installation with an increased max-pods, add a kubelet config file into a k3s, we will use `/etc/rancher/k3s/kubelet.config` :
 * Edit (or create) `/etc/rancher/k3s/kubelet.config`:
      * ```bash
        MAXPOD=250 # the number of pods that you want
        cat > /etc/rancher/k3s/kubelet.config <<EOF
        apiVersion: kubelet.config.k8s.io/v1beta1
        kind: KubeletConfiguration
        maxPods: $MAXPOD
        EOF
        ```
      
 * Edit `/etc/systemd/system/k3s.service` to change the k3s server args:
      * ```
        ExecStart=/usr/local/bin/k3s \
            server \
                '--kubelet-arg=config=/etc/rancher/k3s/kubelet.config'
        ```
      * ⚠️ **If you have any line after `server \` you may want to keep them**
         
 * Reload systemctl to pick up the service change:
      * ```
        sudo systemctl daemon-reload
        ```
        
 * Restart k3s:
      * ```
        sudo systemctl restart k3s
        ```
