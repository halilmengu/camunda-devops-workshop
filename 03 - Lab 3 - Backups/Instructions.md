## Deploy MinIO

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: minio
  name: minio
  namespace: default # Change this value to match the namespace metadata.name
spec:
  containers:
  - name: minio
    image: quay.io/minio/minio:latest
    command:
    - /bin/bash
    - -c
    args: 
    - minio server /data --console-address :9090
    volumeMounts:
    - mountPath: /data
      name: localvolume # Corresponds to the `spec.volumes` Persistent Volume
 # nodeSelector:
 #   kubernetes.io/hostname: kubealpha.local # Specify a node label associated to the Worker Node on which you want to deploy the pod.
  volumes:
  - name: localvolume
 #   hostPath: # MinIO generally recommends using locally-attached volumes
 #     path: /mnt/disk1/data # Specify a path to a local drive or volume on the Kubernetes worker node
 #     type: DirectoryOrCreate # The path to the last directory must exist
---
apiVersion: v1
kind: Service
metadata:
  name: minio
  namespace: default
spec:
  selector:
    app: minio
  ports:
    - protocol: TCP
      port: 9000
      targetPort: 9000
```

## Port-Forward minio

kubectl port-forward pod/minio 9090:9090

## Create Credentials + Bucket

un: minioadmin

pw: minioadmin

## Define Backup Location

```
zeebe:
  affinity:
    podAntiAffinity: null
  clusterSize: 1
  partitionCount: 1
  replicationFactor: 1
  pvcSize: 3Gi
  resources:
    requests:
      cpu: "100m"
      memory: "1Gi"
    limits:
      cpu: "512m"
      memory: "3Gi"
  env:
  - name: ZEEBE_BROKER_DATA_BACKUP_STORE
    value: "S3"
  - name: ZEEBE_BROKER_DATA_BACKUP_S3_BUCKETNAME
    value: "c8-backup"
  - name: ZEEBE_BROKER_DATA_BACKUP_S3_FORCEPATHSTYLEACCESS
    value: "true"
  - name: ZEEBE_BROKER_DATA_BACKUP_S3_ENDPOINT
    value: "http://minio.default.svc.cluster.local:9000"
  - name: ZEEBE_BROKER_DATA_BACKUP_S3_ACCESSKEY
    value: "iQj4X0LD9hTM9sgyD8dx"
  - name: ZEEBE_BROKER_DATA_BACKUP_S3_SECRETKEY
    value: "lgJQ9ucQOFstQ1OObbYVKTl5H0wzTNgFxCZpDS89"
  - name: ZEEBE_BROKER_DATA_BACKUP_S3_REGION
    value: "us-east-1"
```

## Port-Forward Gateway

kubectl port-forward svc/camunda-platform-zeebe-gateway -p 9600:9600


## Pause Exporter

POST localhost:9600/actuator/exporting/pause

## Trigger Backup

POST http://localhost:9600/actuator/backups

{
	"backupId": 1
}

## Get Status of Backup

GET http://localhost:9600/actuator/backups/1

## Resume Exporter

POST localhost:9600/actuator/exporting/resume