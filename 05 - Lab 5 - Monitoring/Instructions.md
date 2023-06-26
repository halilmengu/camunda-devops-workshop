## Installing Prometheus and Grafana

Also for Prometheus we can use a Helm Chart

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack --set prometheus-node-exporter.hostRootFsMount.enabled=false --set grafana.defaultDashboardsTimezone=browser
```

https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack

kubectl port-forward svc/prometheus-kube-prometheus-prometheus 9090:9090

Run query e.g. `apiserver_request_total`

kubectl port-forward svc/prometheus-grafana 8080:80

un: admin
pw: prom-operator

## Import Dashboard

https://github.com/camunda/zeebe/blob/main/monitor/grafana/zeebe.json

--> Nothing to see yet

## Adjust Values

```
prometheusServiceMonitor:
  enabled: true
  labels:
    release: prometheus
  scrapeInterval: 10s
```

## Start

helm install camunda-platform camunda/camunda-platform -f camunda-platform-monitoring-values.yaml 

## Wait

kubectl get pods -w


## View Grafana Dasbhoard again

## Optimize

- Create Multi Process Report
- Discuss differences between monitoring the platform & monitoring processes

## Discussion
- How do participants normally monitor?
- Also Business Reporting?