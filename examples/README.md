
Replace `{{HOST}}` and `{{EMAIL}}` by correct value 

From root of the project:

```bash
kubectl create namespace cert-manager
kubectl apply -f ./sources/cert-manager.v0.11.0.yaml
sleep 120
kubectl apply -f ./example/letsencrypt.yaml
kubectl -n nginx-test apply -f ./example/nginx.yaml
```
