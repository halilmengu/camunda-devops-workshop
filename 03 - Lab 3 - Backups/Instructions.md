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

Go to _Access Keys_ -> create one, download created credentials

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

## Restore from backup

>Disclaimer: This restore procedure or not officially documented and has its flaws. If you need to restore data, please approach us and get up-to-date information.

To be able to restore from a backup, zeebe contains an additional init container in the attached values file.

This init container contains the flag `ZEEBE_RESTORE` are environment variable to control whether a restore procedure should be applied as well as a property `ZEEBE_RESTORE_FROM_BACKUP_ID` that contains the id of the backup to restore.

The behavior is:

* if the zeebe data is empty, the restore of the given backup id will be carried out
* if there is zeebe data, the restore will fail (which will block zeebe from starting)

So, only enable this flag if a restore should be carried out and make sure the zeebe data disk is empty.

>Warning: If a pod fails and is restarted, the init container is running again. If the restore flag is still enabled but data is restored already, the init container will fail.

### Simulate data loss

Uninstall the platform:

```shell
helm uninstall camunda-platform
```

Then, delete all zeebe disks:

```shell
kubectl delete pvc data-camunda-platform-zeebe-0 data-camunda-platform-zeebe-1 data-camunda-platform-zeebe-2
```

The data is now lost!

>Of course, the Elasticsearch data is still present.

### Enable the restore mechanism

Adjust the values of [`camunda-platform-backup-values.yaml`](./camunda-platform-backup-values.yaml):

* `ZEEBE_RESTORE`: `"true"`
* `ZEEBE_RESTORE_FROM_BACKUP_ID`: `"1"` (which is equal to the id provided when taking a backup)

### Install the platform

```shell
helm install camunda-platform camunda/camunda-platform -f camunda-platform-backup-values.yaml
```

Then, you can inspect the logs of the `zeebe-restore` init container:

```shell
kubectl logs camunda-platform-zeebe-0 -c zeebe-restore -f
```

```shell
kubectl logs camunda-platform-zeebe-1 -c zeebe-restore -f
```

```shell
kubectl logs camunda-platform-zeebe-2 -c zeebe-restore -f
```

It reflects that the restore from the backup has been successful.

## Assert that the restore did work out

Remember the process instance from above? It should still have an open user task.

Port-forward Tasklist:

```shell
kubectl port-forward svc/camunda-platform-tasklist 8082:80
```

Then, visit [`localhost:8082`](http://localhost:8082) and complete the open user task.

Next, you can port-forward Operate:

```shell
kubectl port-forward svc/camunda-platform-operate 8081:80
```

Then, visit [`localhost:8081`](http://localhost:8081) and check whether the user task is complete.
