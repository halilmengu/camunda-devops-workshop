## Adjust Values
Using a lower version, so we can update to a newer one.

````
global:
  image:
    tag: 8.2.5
````

````
zeebe-gateway:
  replicas: 1
  affinity:
    podAntiAffinity: null
````

## Install 

helm install camunda-platform camunda/camunda-platform -f camunda-platform-update-values.yaml

## Check Version

kubectl port-forward svc/camunda-plaform-operate  8081:80

Webapp Bottom right

## Change Values

````
global:
  image:
    tag: 8.2.7
````

## Review Creation Order

Zeebe STS 
````
kubectl describe sts ccsm-zeebe
````

````
Events:
  Type    Reason            Age                    From                    Message
  ----    ------            ----                   ----                    -------
  Normal  SuccessfulCreate  4m36s                  statefulset-controller  create Claim data-ccsm-zeebe-0 Pod ccsm-zeebe-0 in StatefulSet ccsm-zeebe success
  Normal  SuccessfulCreate  4m36s                  statefulset-controller  create Claim data-ccsm-zeebe-1 Pod ccsm-zeebe-1 in StatefulSet ccsm-zeebe success
  Normal  SuccessfulCreate  4m36s                  statefulset-controller  create Claim data-ccsm-zeebe-2 Pod ccsm-zeebe-2 in StatefulSet ccsm-zeebe success
  Normal  SuccessfulDelete  2m33s                  statefulset-controller  delete Pod ccsm-zeebe-2 in StatefulSet ccsm-zeebe successful
  Normal  SuccessfulCreate  2m29s (x2 over 4m36s)  statefulset-controller  create Pod ccsm-zeebe-2 in StatefulSet ccsm-zeebe successful
  Normal  SuccessfulDelete  88s                    statefulset-controller  delete Pod ccsm-zeebe-1 in StatefulSet ccsm-zeebe successful
  Normal  SuccessfulCreate  82s (x2 over 4m36s)    statefulset-controller  create Pod ccsm-zeebe-1 in StatefulSet ccsm-zeebe successful
  Normal  SuccessfulDelete  22s                    statefulset-controller  delete Pod ccsm-zeebe-0 in StatefulSet ccsm-zeebe successful
  Normal  SuccessfulCreate  18s (x2 over 4m36s)    statefulset-controller  create Pod ccsm-zeebe-0 in StatefulSet ccsm-zeebe successful
````

  Zeebe Gateway

  ````
  kubectl describe deploy ccsm-zeebe-gateway
  ````

  ````
  Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  4m16s  deployment-controller  Scaled up replica set ccsm-zeebe-gateway-6c5f9767c4 to 1
  Normal  ScalingReplicaSet  2m14s  deployment-controller  Scaled up replica set ccsm-zeebe-gateway-8656495b6b to 1
  Normal  ScalingReplicaSet  43s    deployment-controller  Scaled down replica set ccsm-zeebe-gateway-6c5f9767c4 to 0 from 1
    ````