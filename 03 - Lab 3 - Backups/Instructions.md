## Deploy MinIO

```shell
kubectl apply -f minio-dev.yaml
```

## Port-Forward minio

```shell
kubectl port-forward pod/minio 9090:9090
```

## Create Credentials + Bucket

Login:

username: minioadmin

password: minioadmin

Go to _Access Keys_ -> create one, download created credentials

Go to _Buckets_ -> create one, name it `c8-backup`

## Define Backup Location

Adjust the values of [`camunda-platform-backup-values.yaml`](./camunda-platform-backup-values.yaml):

`ZEEBE_BROKER_DATA_BACKUP_S3_BUCKETNAME`: The name of the bucket you created.
`ZEEBE_BROKER_DATA_BACKUP_S3_ACCESSKEY`: `accessKey` of the created credentials.
`ZEEBE_BROKER_DATA_BACKUP_S3_SECRETKEY`: `secretKey` of the created credentials.

```shell
helm install camunda camunda/camunda-platform -f camunda-platform-backup-values.yaml
```

## Port-Forward Gateway

```shell
kubectl port-forward svc/camunda-platform-zeebe-gateway -p 9600:9600
```

## Pause Exporter

```shell
curl -X POST 'localhost:9600/actuator/exporting/pause'
```

## Trigger Backup

```shell
curl -X POST 'http://localhost:9600/actuator/backups' --header 'Content-Type: application/json' -d '{"backupId": 1}'
```

## Get Status of Backup

```shell
curl 'http://localhost:9600/actuator/backups/1'
```
Wait until the `state` is `COMPLETED` 

## Resume Exporter

```shell
curl -X POST localhost:9600/actuator/exporting/resume
```
