# Kubernetes_setup

Install k3s, create namespace for certs and install certs-manager:

Tested on: Debian: 9, ovh-vps

```
curl -sfL https://get.k3s.io |  sh -
kubectl create namespace cert-manager
kubectl apply -f ./sources/cert-manager.v0.11.0.yaml
sleep 120
```

Edit `./sources/letsencrypt` and change `{{EMAIL}}`

```
kubectl apply -f ./sources/letsencrypt.yaml
```

Config your app inside a namespace `{{APP_NAMESPACE}}`, then edit `./sources/ingress.yaml` and change `{{HOST}}`, `{{PORT}}`, `{{NAME}}` and `{{SERVICE}}`

```
kubectl -n {{APP_NAMESPACE}} apply -f ./sources/ingress.yaml
```

/!\ Caution: your ingress have to be in the same namespace as your app

