The example project for StringBoot service

<div align="center">
    <img src="./assets/images/spring_boot_icon.png"/>
</div>

## Getting Started

## Project structure
```
.
├── spring-boot-prometheus-grafana
│   ├── Dockerfile
│   ...
├── docker-compose.yaml
|
└── README.md
```

## Prerequisites
- Make sure that you have Docker and Docker Compose installed
  - Windows or macOS:
    [Install Docker Desktop](https://www.docker.com/get-started)
  - Linux: [Install Docker](https://www.docker.com/get-started) and then
    [Docker Compose](https://github.com/docker/compose)

## Start project
### Start project in local

- Install infrastructure

- Build project
```shell script
$ ./mvnw clean package
$ ./mvnw spring-boot:run
...
```

### Start project in docker 

- Create volume 
```shell script
$ docker volume create grafana-storage
```

- Start project
```shell script
$ docker-compose up -d --scale spring-boot-prometheus-grafana=2
```

- Stop project
```shell script
$ docker-compose down
```

## Run testing

- Test actuator
```shell script
$ curl -X GET http://localhost:8081/actuator
{"_links":{"self":{"href":"http://localhost:8081/actuator","templated":false},"health":{"href":"http://localhost:8081/actuator/health","templated":false},"health-path":{"href":"http://localhost:8081/actuator/health/{*path}","templated":true},"prometheus":{"href":"http://localhost:8081/actuator/prometheus","templated":false}}}
```

- Metrics of promethues
```shell script
$ curl -X GET http://localhost:8081/actuator/prometheus
...
http_server_requests_seconds_max{exception="None",method="GET",outcome="SUCCESS",status="200",uri="/actuator/prometheus",} 0.2230911
http_server_requests_seconds_max{exception="None",method="GET",outcome="SUCCESS",status="200",uri="/actuator",} 0.2658599
http_server_requests_seconds_max{exception="None",method="GET",outcome="CLIENT_ERROR",status="404",uri="/**",} 0.033111
# HELP process_cpu_usage The "recent cpu usage" for the Java Virtual Machine process
# TYPE process_cpu_usage gauge
process_cpu_usage 0.009735744089012517
...
```

- Send sample request
```shell script
$ sh send_request_script.sh
```

- To see Prometheus dashboard, navigate your browser to http://localhost:9090

- Visit http://localhost:3000 to login Grafana dashboard.

- Setup Prometheus datasource
![setup ds prometheus](./assets/images/setup_ds_prometheus.png)

- Import dashboard id [4701](https://grafana.com/grafana/dashboards/4701) (or load from file spring-boot-metrics.json trong thư mục./assets)
![jvm_1](./assets/images/import_jvm_metric_1.png)
![jvm_2](./assets/images/import_jvm_metric_2.png)

 
- See some jvm metrics

![jmv metrics](assets/images/grafana_jvm_metrics.png)

## Tạo một số metrics

### Success rate for all api
- Metric 1
```text
sum(increase(http_server_requests_seconds_bucket{application="$application", instance="$instance", status="200"}[1m]))
/
sum(increase(http_server_requests_seconds_bucket{application="$application", instance="$instance"}[1m]))
```

- Metric 2
```text
sum(increase(http_server_requests_seconds_bucket{application="$application", instance="$instance", status!="200"}[1m]))
/
sum(increase(http_server_requests_seconds_bucket{application="$application", instance="$instance"}[1m]))
```


- Metric B
```text
sum(increase(http_server_requests_seconds_bucket{uri="/greeting", status!="200"}[1m]))
/
sum(increase(http_server_requests_seconds_bucket{uri="/greeting"}[1m]))
```

### Throughput
```text
sum(rate(http_server_requests_seconds_bucket{application="$application", instance="$instance"}[1m])) by (uri)
```

### Latency P99
```text
histogram_quantile(0.99, sum(rate(http_server_requests_seconds_bucket{application="$application", instance="$instance"}[1m])) by (le, uri))
```

![extra metrics](./assets/images/grafana_extra_metrics.png)

## Contribute

## Reference