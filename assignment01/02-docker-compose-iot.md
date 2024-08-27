# IoT Docker compose


## How to start docker compose

    docker compose up -d

## Error we found

    1. Mongo DB Version is not usable in the gateway server
    2. grafana build needed internet to download package what first startup docker

## How to solve the problems.

    1. ปรับแก้ Version ใน docker.yml file!
    และแก้เครื่องมือที่ยังใช้ไม่ได้โดยการไป Comment ใน docker.yml file!
    2. comment grafana line that said "GF_INSTALL_PLUGINS"

## Output

- [✔] IoT Sensor - Dashboards - Grafana 
- [✔] UI for Apache Ka
- [✔] Mongo Expr
- [✔] Node Expor
- [✔] Prometheus Time Series Collection and Processing Ser
- [✔] Prometheus Pushgateway
- [✔] ZooNavigator


### IoT Sensor - Dashboards - Grafana URL

    172.16.46.21:8085

### UI for Apache Kafka

### Mongo Express

### Node Exporter

### Prometheus Time Series Collection and Processing Server

### Prometheus Pushgateway

### ZooNavigator