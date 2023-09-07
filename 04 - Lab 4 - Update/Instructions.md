## Adjust Values
Using a lower version, so we can update to a newer one.

```
global:
  image:
    tag: 8.2.5
```

To allow running on a single node, all `zeebe` and `zeebe-gateway` pods do get the pre-configured `podAntiAffinity` removed:

```
zeebe-gateway:
  replicas: 1
  affinity:
    podAntiAffinity: null
```

```
zeebe:
  replicas: 1
  affinity:
    podAntiAffinity: null
```

## Install 

```shell
helm install camunda-platform camunda/camunda-platform -f camunda-platform-update-values.yaml
```

## Check Version

```shell
kubectl port-forward svc/camunda-plaform-operate  8081:80
```

Webapp Bottom right

## Change Values

```
global:
  image:
    tag: 8.2.7
```

## Apply the updated values

```shell
helm upgrade camunda-platform camunda/camunda-platform -f camunda-platform-update-values.yaml
```

## Review Creation Order

Zeebe STS 
```shell
kubectl describe sts camunda-platform-zeebe
```

```
Events:
  Type    Reason            Age                    From                    Message
  ----    ------            ----                   ----                    -------
  Normal  SuccessfulCreate  4m36s                  statefulset-controller  create Claim data-camunda-platform-zeebe-0 Pod camunda-platform-zeebe-0 in StatefulSet camunda-platform-zeebe success
  Normal  SuccessfulCreate  4m36s                  statefulset-controller  create Claim data-camunda-platform-zeebe-1 Pod camunda-platform-zeebe-1 in StatefulSet camunda-platform-zeebe success
  Normal  SuccessfulCreate  4m36s                  statefulset-controller  create Claim data-camunda-platform-zeebe-2 Pod camunda-platform-zeebe-2 in StatefulSet camunda-platform-zeebe success
  Normal  SuccessfulDelete  2m33s                  statefulset-controller  delete Pod camunda-platform-zeebe-2 in StatefulSet camunda-platform-zeebe successful
  Normal  SuccessfulCreate  2m29s (x2 over 4m36s)  statefulset-controller  create Pod camunda-platform-zeebe-2 in StatefulSet camunda-platform-zeebe successful
  Normal  SuccessfulDelete  88s                    statefulset-controller  delete Pod camunda-platform-zeebe-1 in StatefulSet camunda-platform-zeebe successful
  Normal  SuccessfulCreate  82s (x2 over 4m36s)    statefulset-controller  create Pod camunda-platform-zeebe-1 in StatefulSet camunda-platform-zeebe successful
  Normal  SuccessfulDelete  22s                    statefulset-controller  delete Pod camunda-platform-zeebe-0 in StatefulSet camunda-platform-zeebe successful
  Normal  SuccessfulCreate  18s (x2 over 4m36s)    statefulset-controller  create Pod camunda-platform-zeebe-0 in StatefulSet camunda-platform-zeebe successful
```

Zeebe Gateway

```shell
kubectl describe deployment camunda-platform-zeebe-gateway
```

```
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  4m16s  deployment-controller  Scaled up replica set camunda-platform-zeebe-gateway-6c5f9767c4 to 1
  Normal  ScalingReplicaSet  2m14s  deployment-controller  Scaled up replica set camunda-platform-zeebe-gateway-8656495b6b to 1
  Normal  ScalingReplicaSet  43s    deployment-controller  Scaled down replica set camunda-platform-zeebe-gateway-6c5f9767c4 to 0 from 1
```
