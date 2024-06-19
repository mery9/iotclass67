# IoT Docker compose


## How to start docker compose

    docker compose up

## Error we found

    เจอ Premission Denied

## How to solve the problems.

    Run Command ตามด้านล่างนี้

    sudo groupadd docker
    sudo usermod -aG docker $USER
    newgrp docker
    docker run hello-world

## Output

- [✔] IoT Sensor - Dashboards - Grafana 
- [✔] UI for Apache Ka
- [✔] Mongo Expr
- [✔] Node Expor
- [✔] Prometheus Time Series Collection and Processing Ser
- [✔] Prometheus Pushgateway
- [✔] ZooNavigator


### IoT Sensor - Dashboards - Grafana URL

### UI for Apache Kafka

### Mongo Express

### Node Exporter

### Prometheus Time Series Collection and Processing Server

### Prometheus Pushgateway

### ZooNavigator