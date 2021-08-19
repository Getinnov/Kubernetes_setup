
#### This folder contain:

* `ingress-example.yaml`

  An *ingress* file enabling to link an existing **application** to the reverse proxy
  you will have to change the value of:

  * `{{NAME}}`: the ingress **name** (anything but an existing one)
  * `{{DOMAIN}}`: the **domain name** used to link to your app
  * `{{Service}}`: an existing **service**
  * `{{PORT}}`: your **service's port**

* `kustomization.yaml` and `nginx-test`

  File and folder to deploy an **nginx app** on a specified **domain** in the namespace `nginx-test`
  you will have to change the value of:

  * `{{DOMAIN}}`: the **domain name** used to link to the nginx app (`./nginx-test/ingress.yaml`)

  To launch the app use, from this directory:

  ```
  kubectl apply -k ./
  ```
