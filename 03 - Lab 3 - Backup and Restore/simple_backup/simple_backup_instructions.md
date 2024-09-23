## Deploy MinIO

```shell
kubectl apply -f minio-dev.yaml
```

## Port-Forward minio

```shell
kubectl port-forward pod/minio 9090:9090
```

## Create Credentials + Bucket

Login: minioadmin / minioadmin

Go to _Access Keys_ -> create one with option "Restrict beyond user policy" enabled, download created credentials

Go to _Buckets_ -> create one, name it `c8-backup`

## Define Backup Location

Adjust the values of [`camunda-platform-backup-values.yaml`](./camunda-platform-backup-values.yaml):

* `ZEEBE_BROKER_DATA_BACKUP_S3_BUCKETNAME`: The name of the bucket you created.
* `ZEEBE_BROKER_DATA_BACKUP_S3_ACCESSKEY`: `accessKey` of the created credentials.
* `ZEEBE_BROKER_DATA_BACKUP_S3_SECRETKEY`: `secretKey` of the created credentials.

Please make sure to adjust them in both positions: the `zeebe.env` and the `zeebe.initContainers[0].env`.

```shell
helm install camunda-platform camunda/camunda-platform -f camunda-platform-backup-values.yaml
```

## Create some data to restore

Port-forward the zeebe gateway once again:

```shell
kubectl port-forward svc/camunda-platform-zeebe-gateway 26500:26500
```

Deploy the created process definition from the last exercise and start a process instance.

Inspect Operate to make sure the process is deployed and an instance is running.

```shell
kubectl port-forward svc/camunda-platform-operate 8081:80
```

Now, we are ready to back up the current zeebe state.

>Please do not perform any action on platform, as we will not restore Elasticsearch.

## Create a backup

### Port-Forward Gateway

```shell
kubectl port-forward svc/camunda-platform-zeebe-gateway 9600:9600
```

### Pause Exporter

```shell
curl -X POST 'localhost:9600/actuator/exporting/pause'
```

### Trigger Backup

```shell
curl -X POST 'http://localhost:9600/actuator/backups' --header 'Content-Type: application/json' -d '{"backupId": 1}'
```

### Get the status of the backup

```shell
curl 'http://localhost:9600/actuator/backups/1'
```

Wait until the `state` is `COMPLETED`

### Resume Exporter

```shell
curl -X POST 'localhost:9600/actuator/exporting/resume'
```

>This command can be omitted to make sure that the Elasticsearch state is in sync with the zeebe state on restore

### Review the backup data

Again, port-forward the minio console:

```shell
kubectl port-forward pod/minio 9090:9090
```

Go to _Buckets_ -> click the bucket `c8-backup` -> click the "folder" icon on the top right

Now, you should be able to see a folder for each partition (1,2 and 3) each containing a folder called "1" in the file browser. Here, you can inspect the data backed up by zeebe.
