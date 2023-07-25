## Download Values.yaml
https://github.com/camunda/camunda-platform-helm/blob/main/kind/camunda-platform-core-kind-values.yaml 

## Start
helm repo add camunda https://helm.camunda.io

helm install camunda-platform camunda/camunda-platform -f camunda-platform-core-kind-values.yaml 

## Wait

kubectl get pods -w

## View Operate
kubectl port-forward svc/camunda-platform-operate  8081:80
http://localhost:8081/


login: demo / demo

## Deploy Diagram
kubectl port-forward svc/camunda-platform-zeebe-gateway 26500:26500 -n default
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

helm delete camunda-platform

kubectl delete pvc

## Optional: Deploy Full Stack

Requirements with default values:
- 3 Nodes
