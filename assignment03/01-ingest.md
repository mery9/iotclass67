# Ingest and store real-time data from IoT sensors


IoT sensors ส่งข้อมูลด้วย MQTT protocol. 
- **Mosquitto (MQTT Broker)**: The MQTT broker (Mosquitto) receives sensor data and publishes it to specified topics.
- **Mosquitto (MQTT Broker)**: MQTT Broker (Mosquitto) ทำหน้าที่รับข้อมูลจากเซนเซอร์และเผยแพร่ไปยังหัวข้อ (topic) ที่กำหนดไว้

- **Kafka Connect MQTT Source**: Kafka Connect pulls data from the MQTT broker and sends it to the Kafka cluster for processing. This integration ensures that raw sensor data is continuously streamed into Kafka topics.
- **Kafka Connect MQTT Source**: Kafka Connect ทำหน้าที่ดึงข้อมูลจาก MQTT Broker และส่งไปยัง Kafka Cluster เพื่อประมวลผล โดยการทำงานร่วมกันแบบนี้จะช่วยให้ข้อมูลดิบจากเซนเซอร์ถูกสตรีมเข้ามาในหัวข้อของ Kafka อย่างต่อเนื่อง

- **Kafka Topics**: Data from IoT sensors is published to Kafka topics (e.g., `iot-frames`, `iot-aggregate-metrics-sensor`).
- **Kafka Topics**: ข้อมูลจากเซนเซอร์ IoT ถูกส่งไปยัง Kafka Topics (เช่น iot-frames, iot-aggregate-metrics-sensor)

- **Kafka Connect MongoDB**: Data stored in Kafka is further processed and consolidated into MongoDB, a NoSQL database. This enables efficient storage of sensor data for long-term access and later viewing.
- **Kafka Connect MongoDB**: ข้อมูลที่ถูกเก็บไว้ใน Kafka จะถูกประมวลผลต่อและรวบรวมไว้ใน MongoDB ซึ่งเป็นฐานข้อมูลแบบ NoSQL ทำให้สามารถจัดเก็บข้อมูลจากเซนเซอร์ได้อย่างมีประสิทธิภาพเพื่อการเข้าถึงและการดูย้อนหลังในระยะยาว




## iot-sensor-1

- IoT Sensor 1 เป็น IoT ที่ได้มาจากการ compose up docker ใน Server ตัวมันเอง

## iot-sensor-2

- IoT Sensor 2 เป็น IoT ที่ได้มาจากการ compose up docker ใน PC ตัวนอกที่ได้ทำการส่งข้อมูลไปยัง Server

## iot-sensor-3-10

- IoT Sensor 3 เป็น IoT ที่ได้ข้อมูลมาจากการที่ IoT เราเอง 1 ตัว และ ตัวอื่นหรือกลุ่มอื่น ๆ ส่งมาให้เราอีก

