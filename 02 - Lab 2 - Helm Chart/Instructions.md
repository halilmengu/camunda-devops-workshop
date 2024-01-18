## Download Values.yaml
https://github.com/camunda/camunda-platform-helm/blob/main/kind/camunda-platform-core-kind-values.yaml 

## Start
```shell
helm repo add camunda https://helm.camunda.io
```

```shell
helm install camunda-platform camunda/camunda-platform -f camunda-platform-core-kind-values.yaml 
```

## Wait

```shell
kubectl get pods -w
```

## View Operate

```shell
kubectl port-forward svc/camunda-platform-operate  8081:80
```

Go to http://localhost:8081/

Login: demo / demo

## View Tasklist

```shell
kubectl port-forward svc/camunda-platform-tasklist  8082:80
```

Go to http://localhost:8082/

Login: demo / demo

## Deploy Diagram

```shell
kubectl port-forward svc/camunda-platform-zeebe-gateway 26500:26500
```

Endpoint: http://localhost:26500
- Include User Task
- Include Connector (e.g. REST https://httpbin.org/)

## Start Instance
- Go to Operate
kubectl port-forward svc/camunda-platform-tasklist 8082:80
- Go to Tasklist

## Review values.yaml
- Talk about Limitations
    - 1 Broker
    - No Identity
    - No Optimize
- Upgrades / Updates

## Delete Release

```shell
helm uninstall camunda-platform
```

```shell
kubectl delete pvc --all
```

## Optional: Deploy Full Stack

Requirements with default values:
- 3 Nodes
