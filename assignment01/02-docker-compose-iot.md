# IoT Docker compose
```
volumes:
    prometheus_data: {}
    grafana_data: {}
    zookeeper-data:
      driver: local
    zookeeper-log:
      driver: local
    kafka-data:
      driver: local

services:

  zookeeper:
    # ZooKeeper ทำหน้าที่จัดการ Kafka brokers, ตรวจสอบสถานะโหนด, และจัดการ leader election เมื่อโหนดล้มเหลว ทำให้คลัสเตอร์ Kafka ทำงานได้อย่างเสถียรและมีประสิทธิภาพ
    image: confluentinc/cp-zookeeper
    container_name: zookeeper
    # ZooKeeper is designed to "fail-fast", so it is important to allow it to
    # restart automatically.
    restart: unless-stopped
    volumes:
      - zookeeper-data:/var/lib/zookeeper/data
      - zookeeper-log:/var/lib/zookeeper/log
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_LOG4J_ROOT_LOGLEVEL: INFO
      ZOOKEEPER_LOG4J_PROP: INFO,ROLLINGFILE
      ZOOKEEPER_LOG_MAXFILESIZE: 10MB
      ZOOKEEPER_LOG_MAXBACKUPINDEX: 10
      ZOOKEEPER_SNAP_COUNT: 10
      ZOOKEEPER_AUTOPURGE_SNAP_RETAIN_COUNT: 10
      ZOOKEEPER_AUTOPURGE_PURGE_INTERVAL: 3


  kafka:
    # Kafka เป็น Platform การจัดการข้อมูล Streaming ที่ใช้ในการ ส่งข้อความ (messaging) และ ประมวลผลข้อมูลแบบ Real-time โดย Kafka นั้นได้มีการออกแบบมาเพื่อรองรับปริมาณข้อมูลขนาดใหญ่ที่ต้องการการ
    # กระจายตัวและการประมวลผลแบบต่อเนื่อง จุดเด่นของ Kafka คือความสามารถในการทำงานแบบกระจาย (distributed), รองรับการขยายขนาด (scalable), และการทนต่อความล้มเหลว (fault-tolerant)
    image: confluentinc/cp-kafka
    container_name: kafka
    volumes:
      - kafka-data:/var/lib/kafka
    restart: unless-stopped
    environment:
      # Required. Instructs Kafka how to get in touch with ZooKeeper.
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_NUM_PARTITIONS: 1
      KAFKA_COMPRESSION_TYPE: gzip
      # Required when running in a single-node cluster, as we are. We would be able to take the default if we had
      # three or more nodes in the cluster.
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      # Required. Kafka will publish this address to ZooKeeper so clients know
      # how to get in touch with Kafka. "PLAINTEXT" indicates that no authentication
      # mechanism will be used.
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'
    links:
      - zookeeper


  kafka-rest-proxy:
    # Kafka REST Proxy เป็น RESTful Interface สำหรับ Kafka Cluster
    image: confluentinc/cp-kafka-rest:latest
    container_name: kafka-rest-proxy
    environment:
      # Specifies the ZooKeeper connection string. This service connects
      # to ZooKeeper so that it can broadcast its endpoints as well as
      # react to the dynamic topology of the Kafka cluster.
      KAFKA_REST_ZOOKEEPER_CONNECT: zookeeper:2181
      # The address on which Kafka REST will listen for API requests.
      KAFKA_REST_LISTENERS: http://0.0.0.0:8082/
      # Required. This is the hostname used to generate absolute URLs in responses.
      # It defaults to the Java canonical hostname for the container, which might
      # not be resolvable in a Docker environment.
      KAFKA_REST_HOST_NAME: kafka-rest-proxy
      # The list of Kafka brokers to connect to. This is only used for bootstrapping,
      # the addresses provided here are used to initially connect to the cluster,
      # after which the cluster will dynamically change. Thanks, ZooKeeper!
      KAFKA_REST_BOOTSTRAP_SERVERS: kafka:9092
    # Kafka REST relies upon Kafka, ZooKeeper
    # This will instruct docker to wait until those services are up
    # before attempting to start Kafka REST.
    restart: unless-stopped
    ports:
      - "9999:8082"
    depends_on:
      - zookeeper
      - kafka

  kafka-connect:
    # Kafka Connect คือ Framework ที่ใช้สำหรับการเชื่อมต่อและเคลื่อนย้ายข้อมูล ระหว่างระบบภายนอกและ Kafka โดยไม่จำเป็นต้องเขียนโค้ดเพิ่มเติมให้ซับซ้อน ทำให้สามารถส่งหรือดึงข้อมูลจากแหล่งข้อมูลที่หลากหลาย
    # เช่น ฐานข้อมูล (SQL/NoSQL), ไฟล์, ระบบคลาวด์, และอื่น ๆ ไปยัง Kafka หรือจาก Kafka ไปยังระบบปลายทางอื่น ๆ ได้
    image: confluentinc/cp-kafka-connect:latest
    hostname: kafka-connect
    container_name: kafka-connect
    environment:
      # Required.
      # The list of Kafka brokers to connect to. This is only used for bootstrapping,
      # the addresses provided here are used to initially connect to the cluster,
      # after which the cluster can dynamically change. Thanks, ZooKeeper!
      CONNECT_BOOTSTRAP_SERVERS: "kafka:9092"
      # Required. A unique string that identifies the Connect cluster group this worker belongs to.
      CONNECT_GROUP_ID: kafka-connect-group
      # Connect will actually use Kafka topics as a datastore for configuration and other data. #meta
      # Required. The name of the topic where connector and task configuration data are stored.
      CONNECT_CONFIG_STORAGE_TOPIC: kafka-connect-meta-configs
      # Required. The name of the topic where connector and task configuration offsets are stored.
      CONNECT_OFFSET_STORAGE_TOPIC: kafka-connect-meta-offsets
      # Required. The name of the topic where connector and task configuration status updates are stored.
      CONNECT_STATUS_STORAGE_TOPIC: kafka-connect-meta-status
      # Required. Converter class for key Connect data. This controls the format of the
      # data that will be written to Kafka for source connectors or read from Kafka for sink connectors.
      CONNECT_KEY_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      # Required. Converter class for value Connect data. This controls the format of the
      # data that will be written to Kafka for source connectors or read from Kafka for sink connectors.
      CONNECT_VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      # Required. The hostname that will be given out to other workers to connect to.
      CONNECT_REST_ADVERTISED_HOST_NAME: "kafka-connect"
      CONNECT_REST_PORT: 8083
      # The next three are required when running in a single-node cluster, as we are.
      # We would be able to take the default (of 3) if we had three or more nodes in the cluster.
      CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: "1"
      CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: "1"
      CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: "1"
      #Connectos path
      CONNECT_PLUGIN_PATH: "/usr/share/java,/data/connectors/"
      CONNECT_LOG4J_ROOT_LOGLEVEL: "INFO"
    restart: unless-stopped
    volumes:
      - ./kafka_connect/data:/data
    command: 
      - bash 
      - -c 
      - |
        echo "Launching Kafka Connect worker"
        /etc/confluent/docker/run & 
        #
        echo "Waiting for Kafka Connect to start listening on http://$$CONNECT_REST_ADVERTISED_HOST_NAME:$$CONNECT_REST_PORT/connectors ⏳"
        while [ $$(curl -s -o /dev/null -w %{http_code} http://$$CONNECT_REST_ADVERTISED_HOST_NAME:$$CONNECT_REST_PORT/connectors) -ne 200 ] ; do 
          echo -e $$(date) " Kafka Connect listener HTTP state: " $$(curl -s -o /dev/null -w %{http_code} http://$$CONNECT_REST_ADVERTISED_HOST_NAME:$$CONNECT_REST_PORT/connectors) " (waiting for 200)"
          sleep 5 
        done
        nc -vz $$CONNECT_REST_ADVERTISED_HOST_NAME $$CONNECT_REST_PORT
        echo -e "\n--\n+> Creating Kafka Connect MongoDB sink Current PATH ($$PWD)"
        /data/scripts/create_mongo_sink.sh 
        echo -e "\n--\n+> Creating MQTT Source Connect Current PATH ($$PWD)"
        /data/scripts/create_mqtt_source.sh
        echo -e "\n--\n+> Creating Kafka Connect Prometheus sink Current PATH ($$PWD)"
        /data/scripts/create_prometheus_sink.sh
        sleep infinity
    # kafka-connect relies upon Kafka and ZooKeeper.
    # This will instruct docker to wait until those services are up
    # before attempting to start kafka-connect.
    depends_on:
      - zookeeper
      - kafka


  # Eclipse Mosquitto is an open source (EPL/EDL licensed) message broker that implements the MQTT protocol versions 5.0, 3.1.1 and 3.1. Mosquitto is lightweight and is suitable for use on all devices from low power single board computers to full servers.
  # The MQTT protocol provides a lightweight method of carrying out messaging using a publish/subscribe model. This makes it suitable for Internet of Things messaging such as with low power sensors or mobile devices such as phones, embedded computers or microcontrollers.
  # The Mosquitto project also provides a C library for implementing MQTT clients, and the very popular mosquitto_pub and mosquitto_sub command line MQTT clients.
  mosquitto:
    # Mosquitto เป็น MQTT Broker ที่รองรับ MQTT protocol สำหรับ Messaging ที่มีขนาดเล็ก ใช้พลังงานที่ต่ำ ทำให้เหมาะสำหรับอุปกรณ์ที่มีข้อจำกัดด้านพลังงานหรือ Bandwidth
    # เหมาะสำหรับการใช้งานใน IoT เป็นอย่างยิ่ง
    image: eclipse-mosquitto:latest
    hostname: mosquitto
    container_name: mosquitto
    restart: unless-stopped
    ports:
      - "1883:1883"
      - "9001:9001"
    volumes:
      - ./mosquitto/config:/mosquitto/config

  mongo:
    # MongoDB คือฐานข้อมูลแบบ NoSQL ที่ถูกออกแบบมาให้รองรับการจัดเก็บข้อมูลที่มีความยืดหยุ่นและไม่ต้องใช้โครงสร้างตายตัว (schema-less) จัดเก็บข้อมูลในรูปแบบของเอกสาร (document) ซึ่งใช้โครงสร้าง JSON        # หรือ BSON (Binary JSON) ทำให้สามารถจัดเก็บข้อมูลที่ซับซ้อนและหลากหลายได้ง่ายขึ้น
    image: mongo:4.4.20
    container_name: mongo
    env_file:
      - .env
    restart: unless-stopped
    environment:
      - MONGO_INITDB_ROOT_USERNAME=${MONGO_ROOT_USER}
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_ROOT_PASSWORD}
      - MONGO_INITDB_DATABASE=${MONGO_DB}
      
  grafana:
    # Grafana เป็นเครื่องมือที่ใช้สำหรับการแสดงผลข้อมูล (Visualization) และวิเคราะห์ข้อมูล (Analyze) ผ่านการสร้าง Dashboard แบบ Interactive โดย Grafana สามารถเชื่อมต่อกับแหล่งข้อมูลที่หลากหลาย       # เช่นฐานข้อมูล SQL, NoSQL, หรือแหล่งข้อมูลอื่นๆ เช่น Prometheus, InfluxDB และ Elasticsearch ทำให้ผู้ใช้งานสามารถดึงข้อมูลเรียลไทม์หรือข้อมูลย้อนหลังมาวิเคราะห์ในรูปแบบกราฟหรือตาราง
    image: grafana/grafana:9.5.20-ubuntu
    container_name: grafana
    user: '0'
    volumes:
      - ./grafana/data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources
      - ./grafana/data/plugins:/var/lib/grafana/plugins
    environment:
      - GF_SECURITY_ADMIN_USER=${ADMIN_USER:-admin}
      - GF_SECURITY_ADMIN_PASSWORD=${ADMIN_PASSWORD:-admin}
      - GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-worldmap-panel,grafana-piechart-panel
      - GF_USERS_ALLOW_SIGN_UP=false
    restart: unless-stopped
    links:
       - prometheus
    ports:
      - '8085:3000'
```

## start-service #0
```
docker compose up zookeeper kafka
```
start service ในหน้าจอ service_0 มี zookeeper และ kafka
โดยเราต้องรอให้ทั้ง 2 ตัวนี้ Run ให้เรียบร้อยก่อนที่จะไป Step ต่อไป เพื่อให้เราไม่มีปัญหาในตอนข้อมูลขึ้น
โดยให้รออย่างน้อย 5 นาที หรือ จน terminal นั้นมีความนิ่ง

## start-service #1
```
docker compose up kafka-rest-proxy kafka-connect mosquitto mongo grafana prometheus
```
start service ในหน้าจอ service_1 มี kafka-rest-proxy, kafka-connect, mosquitto, mongo, grafana และ prometheus
โดยเราต้องรอให้ทั้ง 6 ตัวนี้ Run ให้เรียบร้อยก่อนที่จะไป Step ต่อไป เพื่อให้เราไม่มีปัญหาในตอนข้อมูลขึ้น
โดยให้รออย่างน้อย 5 นาที หรือ จน terminal นั้นมีความนิ่ง

## start-service #2
```
docker compose up iot-processor
```
start service ในหน้าจอ service_2 มี iot-processor
โดยเราต้องรอให้ทั้ง 1 ตัวนี้ Run ให้เรียบร้อยก่อนที่จะไป Step ต่อไป เพื่อให้เราไม่มีปัญหาในตอนข้อมูลขึ้น
โดยให้รออย่างน้อย 5 นาที หรือ จน terminal นั้นมีความนิ่ง
หรือจน iot-processor นั้นมีข้อความที่ขึ้นว่า REBLANCEING TO RUNNING เป็นคำสำคัญ
ไม่อย่างงั้นต้องทำการ Restart ไปเรื่อย ๆ

## start-service #3
```
docker compose up iot_sensor_1
```
start service ในหน้าจอ service_3 มี iot_sensor_1
โดยให้ Run ได้เลย
