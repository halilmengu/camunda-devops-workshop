## Adjust Values
Using a lower version, so we can update to a newer one.

```
operate:
  image:
    tag: 8.5.5
```

To allow running on a single node, all `zeebe` and `zeebe-gateway` pods do get the pre-configured `podAntiAffinity` removed:

```
zeebeGateway:
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
operate:
  image:
    tag: 8.5.6
```

## Apply the updated values

```shell
helm upgrade camunda-platform camunda/camunda-platform -f camunda-platform-update-values.yaml
```

## Review Creation Order

Zeebe STS 
```shell
kubectl describe deployment camunda-platform-operate
```

```
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  6m49s  deployment-controller  Scaled up replica set camunda-platform-operate-776f9f9f89 to 1
  Normal  ScalingReplicaSet  69s    deployment-controller  Scaled down replica set camunda-platform-operate-776f9f9f89 to 0 from 1
  Normal  ScalingReplicaSet  68s    deployment-controller  Scaled up replica set camunda-platform-operate-79b674f57f to 1
```
