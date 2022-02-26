```
kubectl create namespace monitoring
kubectl apply -k ./
```

cat - > test <<EOF 
apiVersion: v1
kind: ConfigMap
metadata:
   name: proxy-environment-variables
   namespace: kube-system
data:
   HTTP_PROXY: http://customer.proxy.host:proxy_port
   HTTPS_PROXY: http://customer.proxy.host:proxy_port
   NO_PROXY: localhost,127.0.0.1
EOF
