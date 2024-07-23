# Example Upgrade path for Camunda 8.2 -> 8.5


# Install Camunda 8.2.20 with Elasticsearch 7.17.15 and sample data
Helm Chart used: https://artifacthub.io/packages/helm/camunda/camunda-platform/8.2.20

## Install Camunda 8.2.20
```bash
helm install --namespace camunda camunda camunda/camunda-platform -f ./Camunda8.2.20-with-ES7.17.15-values.yaml --version 8.2.20 --create-namespace
```

## Create sample data

```bash
kubectl create configmap models --from-file=CamundaProcess.bpmn=./resources/BenchmarkProcess.bpmn
```

```bash
kubectl apply -f ./resources/zbctl-deploy-job.yaml 
```

```bash
kubectl logs -f $(kubectl get pods --selector=job-name=zbctl-deploy --output=jsonpath='{.items[*].metadata.name}' | awk '{print $1}') 
```

```bash
kubectl create configmap payload --from-file=./resources/payload.json
```

```bash
kubectl apply -f ./resources/benchmark.yaml
```

```bash
kubectl delete deploy benchmark
```

## Check state in Operate
![Screenshot Operate 8.2.20](images/state-operate-8.2.20.png)


# Upgrade Camunda 8.2.20 to 8.3.3
Docs: https://docs.camunda.io/docs/self-managed/operational-guides/update-guide/820-to-830/  
Docs: https://docs.camunda.io/docs/self-managed/setup/upgrade/#from-camunda-82-to-83

## Update deprecated Elasticsearch properties
```bash
kubectl set env statefulset/elasticsearch-master node.roles='data','ingest','master','remote_cluster_client','ml'

kubectl set env statefulset/elasticsearch-master node.data-
kubectl set env statefulset/elasticsearch-master node.ingest-
kubectl set env statefulset/elasticsearch-master node.master-
kubectl set env statefulset/elasticsearch-master node.remote_cluster_client-
kubectl set env statefulset/elasticsearch-master node.ml-
```

## Data Retention in ES
Option Two: Update PVs manually, see https://docs.camunda.io/docs/self-managed/platform-deployment/helm-kubernetes/upgrade/#elasticsearch---data-retention

```bash
ES_PV_NAME0="$(kubectl get pvc elasticsearch-master-elasticsearch-master-0 -o jsonpath='{.spec.volumeName}')"
ES_PV_NAME1="$(kubectl get pvc elasticsearch-master-elasticsearch-master-1 -o jsonpath='{.spec.volumeName}')"
kubectl patch pv "${ES_PV_NAME0}" -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
kubectl patch pv "${ES_PV_NAME1}" -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
```

```bash
kubectl scale statefulset elasticsearch-master --replicas 0
```

```bash
kubectl delete pvc elasticsearch-master-elasticsearch-master-0
kubectl delete pvc elasticsearch-master-elasticsearch-master-1
```

### Edit elasticsearch-master PVs manually
***MANUAL STEPS SINCE MY SCRIPT BROKE AND COST ME LIKE 5 HOURS. ORIGINAL SCRIPT AT THE BOTTOM, TO-BE-FIXED/AUTOMATED AGAIN***

PV: Delete resourceVersion 
claimRef: Delete resourceVersion, UUID
claimRef: Edit the claimRef to include the name of the new PVCs that will appear after the upgrade.  
--> data-camunda-elasticsearch-master-0  
--> data-camunda-elasticsearch-master-1


## actual upgrade

```bash
kubectl -n camunda delete deployment camunda-operate
kubectl -n camunda delete deployment camunda-tasklist
kubectl -n camunda delete deployment camunda-zeebe-gateway
kubectl -n camunda delete statefulset camunda-zeebe
```

```bash
helm upgrade -f ./Camunda8.3.3-with-ES8.8.2-values.yaml camunda camunda/camunda-platform --version 8.3.3
```


# Upgrade Camunda 8.3.3 to 8.4.0
Docs: https://docs.camunda.io/docs/self-managed/operational-guides/update-guide/830-to-840/  
Docs: https://docs.camunda.io/docs/self-managed/setup/upgrade/#from-camunda-83-to-84

```bash
helm upgrade -f ./Camunda8.4.0-with-ES8.9.2-values.yaml  camunda camunda/camunda-platform --version 9.3.4
```


# Upgrade Camunda 8.4.0 to 8.5.
Docs: https://docs.camunda.io/docs/self-managed/operational-guides/update-guide/840-to-850/  
Docs: https://docs.camunda.io/docs/self-managed/setup/upgrade/#from-camunda-84-to-85


```bash
helm upgrade -f ./Camunda8.5.0-with-ES8.13.0-values.yaml  camunda camunda/camunda-platform --version 10.0.2
```


# Verification
## Operate still shows correct data (verifies Operate/Elasticsearch data is correct)  
![Screenshot Operate 8.5.0 before cancel](images/state-operate-8.5.0-before-cancel.png)
  
## Instance can be cancelled (verifies also Zeebe data is correct and matches Operate/Elasticsearch)  
![Screenshot Operate 8.5.0 after cancel](images/state-operate-8.5.0-after-cancel.png)






## broken scripts:

Make PVs bind to new Elastic PVCs
```bash
kubectl get pv -o yaml | yq -r '.items[] | select(.spec.claimRef != null) | select(.spec.claimRef.name == "elasticsearch-master-elasticsearch-master-0")' > master-0.yaml
yq eval 'del(.metadata.resourceVersion, .spec.claimRef.resourceVersion, .spec.claimRef.uuid, .spec.claimRef.name)' -i master-0.yaml
yq eval '.spec.claimRef.name = "data-camunda-elasticsearch-master-0"' -i master-0.yaml
kubectl apply -f master-0.yaml
rm master-0.yaml
```

```bash
kubectl get pv -o yaml | yq -r '.items[] | select(.spec.claimRef != null) | select(.spec.claimRef.name == "elasticsearch-master-elasticsearch-master-1")' > master-1.yaml
yq eval 'del(.metadata.resourceVersion, .spec.claimRef.resourceVersion, .spec.claimRef.uuid, .spec.claimRef.name)' -i master-1.yaml
yq eval '.spec.claimRef.name = "data-camunda-elasticsearch-master-1"' -i master-1.yaml
kubectl apply -f master-1.yaml
rm master-1.yaml
```