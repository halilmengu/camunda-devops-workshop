## Download Repo

https://github.com/camunda/camunda-platform#using-docker-compose

## Docker-Compose up

```shell
docker compose -f docker-compose-core.yaml up -d
```

## View Operate

Url: http://localhost:8081/

login: demo / demo

## Deploy Diagram

Endpoint: http://localhost:26500

- Include User Task
- Include Connector (e.g. REST https://httpbin.org/)

## Start Instance

Start instance from the modeler

- Go to Operate
- Go to Tasklist

## Review Docker-Compose

- Talk about Limitations
    - 1 Broker
    - No Identity
    - No Optimize
- Upgrades / Updates

## Docker Compose Down

```shell
docker compose -f docker-compose-core.yaml down
```

## Optional: Enable kibana with a profile

To run kibana for es data inspection, use the kibana profile:

```shell
docker compose -f docker-compose-core.yaml --profile kibana up -d
```

## Optional: Show Full Stack

Change in env:

`ZEEBE_AUTHENTICATION_MODE=identity`

```shell
docker compose up -d
```
(will run with `docker-compose.yaml`)
