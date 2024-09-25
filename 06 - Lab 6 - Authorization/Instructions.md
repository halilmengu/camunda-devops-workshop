# Camunda OIDC Integration

Follow these instructions to setup a local Camunda 8 cluster with an external OIDC provider.

We will use a Keycloak that is installed separately of Camunda as our OIDC provider.

## Keycloak Setup

Install Keycloak using bitnami's Helm chart:
```
helm install camunda-keycloak oci://registry-1.docker.io/bitnamicharts/keycloak
```

Wait for Keycloak to start:
```
kubectl get pods -w
```

Grap the admin users password:
```
kubectl get secret camunda-keycloak -o jsonpath={.data.admin-password} | base64 -d
```

Port forward Keycloak
```
kubectl port-forward svc/camunda-keycloak 18080:80
```

Configure the Camunda realm:
1. Sign in to Keycloak as `user` with password extracted above
2. Create a new Realm called `camunda`
3. Create a user with name `demo` and set their password to `demo`
4. Create the following client `identity-api`, `zeebe-api`, `operate-api`, `tasklist-api`, and `optimize-api`
    - Create an OpenID Connect client
    - Activate Client-Authentication with Service Account Authentication Flow
    - To keep things simple add `*` as the redirection URL
5. For each client add adapt the scope
    - Naviagte to Client Scopes
    - Select the Assigned client scope `<client-id>-dedicated`
    - Add Mapper by Configuration and select Audience
    - Enter the clients ID as name and select it in the client audience

## Camunda Setup

Based on your configuration in Keycloak replace the placeholders in the file `camunda-values.yaml`, i.e., the client secrets and the user id.
```
helm install camunda-platform camunda/camunda-platform -f camunda-values.yaml
```

Use port forwarding to access the Camunda apps:
```
kubectl port-forward svc/camunda-identity 8080:80
kubectl port-forward svc/camunda-operate 8081:80
kubectl port-forward svc/camunda-tasklist 8082:80
kubectl port-forward svc/camunda-optimize 8083:80
```

If you visit one of the webpages and you are forwarded to the host `camunda-keycloak` change it manually to `localhost:18080`

## Delete Setup

```
helm uninstall camunda-platform
helm uninstall camunda-keycloak
kubectl delete pvc data-camunda-keycloak-postgresql-0 data-camunda-elasticsearch-master-0 data-camunda-identity-postgresql-0 data-camunda-zeebe-0
``` 
