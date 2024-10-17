# Main technologies of architecture

## Architecture Overview

![IoT Event Streaming Architecture ![image](https://github.com/user-attachments/assets/e0574075-9b00-4868-a49f-54fb8b1b50b0)


## Eclipse Mosquitto

Open Source สำหรับการจัดการ Message (Message Broker) ที่ใช้งาน MQTT Protocol โดย Mosquitto มีขนาดที่เล็กและใช้พลังงานน้อย ทำให้สามารถใช้งานได้บนบอร์ด Embedded System ขนาดเล็ก, คอมพิวเตอร์ขนาดต่างๆ จนถึงเครื่อง Server
MQTT Protocol (Message Queuing Telemetry Transport) เป็น Protocol สำหรับการใช้ส่ง Message ระหว่างอุปกรณ์ (Machine to Machine) แบบ Asynchronous (Pub/Sub Model) เหมาะสำหรับการใช้งานบนระบบ IoT ที่ประกอบด้วย Low Power Sensor และ Mobile Device ในระบบ โดยในการส่ง Message ของ MQTT จะทำการส่งผ่านเครือข่าย TCP/IP ทำให้สามารถเชื่อม Device ต่างๆ เข้าด้วยกันได้ผ่านระบบ Internet ความแตกต่างของ MQTT และ HTTP คือ MQTT ถูกออกแบบมาให้ใช้งานบนอุปกรณืขนาดเล็กทำให้มี Bandwidth ต่ำกว่า HTTP


## Apache ZooKeeper

ระบบ Central Service สำหรับการกำหนดค่าต่างๆ ระหว่าง Service ในกลุ่มเดียวกัน เพื่อลดความซับซ้อนในการใช้งานขณะที่ทำการ Deploy application ช่วยให้การจัดการระบบกระจายเป็นไปอย่างมีประสิทธิภาพ โดยตัวอย่างหน้าที่ของ Zookeeeper มีดังนี้
- การจัดการ Broker และตรวจสอบสถานะของ Broker
- Topic reccording และ Topic partition
- Update Kafka
- Topic Authorization
- และอื่นๆ

## Apache Kafka

Apache Kafka เป็น Platform สำหรับการใช้ส่ง, เก็บ, ดำเนินการ และติดตาม Data stream จากหลายแหล่งที่มาเหมาะสำหรับการสร้าง data pipeline ที่มีประสิทธิภาพสูงและการวิเคราะห์ข้อมูลแบบ Real Time โดยมีข้อดีคือความสามารถในการจัดการข้อมูลปริมาณมากและมีความยืดหยุ่นสูง 
Apache Kafka ถูกใช้เป็นตัวกลางสำหรับการเชื่อมต่อ เพื่อช่วยแยกกการสื่อสารระหว่างระบบแต่ละตัว 


## Apache Kafka Connect

เป็น Framework ในการเชื่อมต่อเครือข่ายในระบบเข้ากับเรือข่ายภายนอก เช่น Database หรือ File Systems ด้วยการใช้งาน Kafka เป็นตัวกลางในการรับส่ง Data โดยจะเน้นที่ Streaming Data ในการใช้งาน Kafka connect จะต้องมีการสร้าง Connecter ได้แก่ Source connecter สำหรับรับ/อ่านข้อมูล และ Sink Connecter สำหรับส่ง/เขียนข้อมูล เพื่อให้สามารถระบุต้นทางและปลายทางได้


## Apache Kafka Streams หรือ IoT Process
เป็นเครื่องมือที่ทรงพลังสำหรับการประมวลผลข้อมูล โดยเฉพาะอย่างยิ่งในบริบทของ Internet of Things มันเป็น Client Library ที่ช่วยให้ DevOps สามารถสร้าง Application และ Microservice ได้อย่างมีประสิทธิภาพ โดยเน้นที่การประมวลผลและวิเคราะห์ข้อมูลที่อยู่ใน Kafka Cluster

Kafka Streams โดดเด่นในการจัดการกับข้อมูลจำนวนมหาศาลที่เกิดขึ้นในระบบ IoT และสามารถประมวลผลแบบเรียลไทม์ได้อย่างรวดเร็ว ซึ่งเป็นคุณสมบัติที่สำคัญมากในโลกของ IoT ที่ต้องการการตอบสนองแบบว่องไว นอกจากนี้ ความยืดหยุ่นสูงของ Kafka Streams ยังช่วยให้ระบบสามารถปรับขนาดได้ตามความต้องการที่เปลี่ยนแปลง และเพิ่มประสิทธิภาพในการทำงาน ทำให้เหมาะสำหรับการใช้งานในหลากหลายสถานการณ์

## Prometheus

Prometheus เป็นระบบ Mornitoring toolkit แบบ Open-source สำหรับการติดตาม และแจ้งเตือน Event ต่างๆ ของ Database โดยทำงานด้วยการบันทึก Metric ของ Time-series Database ที่ใช้งาน HTTP Pull Model
ส่วนประกอบของ Prometheus มีดังนี้
- Prometheus Server: ทำหน้าที่จัดการข้อมูล Time Series ในการดึง/เก็บข้อมูล
- Client Libraries: ทำหน้าที่จัดรูปแบบข้อมูลในฝั่ง Client ให้เป็น Format ของ Prometheus
- Push Gateway: ทำหน้าที่เป็นตัวกลางในการส่งข้อมูล เพื่อที่จะได้ไม่ต้องทำการ Pull บ่อยเกินไป
- Exporters: ทำหน้าที่แปลงข้อมูลในระบบอื่นๆ ที่มีอยู่ให้อยู่มนรูปของ Matrics ที่ Prometheus สามารถใช้งานได้
- Alert Manager: ทำหน้าที่ส่วนส่วนของ Push Alerts เช่น การส่ง Notification
- Data Visualization and Export: Programs ส่วน UI ที่ดึงข้อมูลมาแสดงผล เช่น Grafana

## MongoDB

MongoDB เป็นระบบ NoSQL Database แบบ Open source ใช้ Document Data Model โดยเก็บแบบ JSON Object สำหรับการจัดเก็บและการจัดการข้อมูล ช่วยให้การจัดเก็บและการจัดการข้อมูลเป็นไปอย่างยืดหยุ่นและมีประสิทธิภาพ ซึ่งมีความสามารถในการขยายและการจัดการข้อมูลที่มีประสิทธิภาพสูง รูปแบบการจัดลำดับไฟล์ของ MongoDB จะแบ่งเป็น 3 ลำดับหลัก คือ Database, Collection และ Document

## Grafana

Open Source Dash Board Tool ที่มีพื้นฐานมาจาก Apache 2.0 ใช้ในการแสดงผลข้อมูลที่อยู่ในรูปแบบ Metric Data สามารถแสดงผลได้ด้วยการสร้าง Dashboard และกราฟจากข้อมูล รองรับการแสดงข้อมูลจาก Time Series Databases ได้แบบ Realtime, มีความยืดหยุ่นในการตั้งค่าการแสดงผลของ Graph และ Panel อีกทั้งใช้งานได้กับหลาย Data source

## IoT Sensor

IoT sensor เป็นอุปกรณ์ที่ใช้ในการรวบรวมข้อมูลสภาพแวดล้อม และส่งค่าไปที่อุปกรณ์อื่นๆ บนระบบเครื่อข่าย หรือระบบ Internet โดยตัวอุปกรณ์จะแบ่งได้หลากหลายประเภทตามการใช้งาน และใน 1 Device สามารถทำได้มากกว่า 1 อย่างเช่น อุณหภูมิ ความสว่าง และความชื้น ซึ่งข้อมูลจาก IoT sensor สามารถใช้ในการ ติดตาม/ตรวจสอบสภาพแวดล้อม การควบคุมระบบต่างๆ เช่น ระบบไฟฟ้าในอาคาร และการวิเคราะห์ข้อมูลต่างๆ เช่นการทำนายสภาพอากาศ 
