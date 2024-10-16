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


## Apache Kafka Streams

เป็น Client Library สำหรับการสร้าง Application และ Microservice ใช้สำหรับการประมวลผลและการวิเคราะห์ข้อมูลที่เก็บใน Kafka Cluster ช่วยให้การสร้าง Application และ Microservice ที่มีประสิทธิภาพสูงและสามารถประมวลผลข้อมูลแบบเรียลไทม์

## Prometheus

เป็น Software สำหรับการติดตามและแจ้งเตือน Event ต่างของ Database โดยทำงานด้วยการบันทึก Metric ของ Time-series Database ที่ใช้งาน HTTP Pull Model

## MongoDB

MongoDB เป็นระบบ NoSQL Database แบบ Open source ใช้ Document Data Model สำหรับการจัดเก็บและการจัดการข้อมูล ช่วยให้การจัดเก็บและการจัดการข้อมูลเป็นไปอย่างยืดหยุ่นและมีประสิทธิภาพ ซึ่งมีความสามารถในการขยายและการจัดการข้อมูลที่มีประสิทธิภาพสูง

## Grafana

Software ที่มีพื้นฐานมาจาก Apache 2.0 ใช้ในการแสดงผลข้อมูลที่อยู่ในรูปแบบ Metric Data สามารถแสดงผลได้ด้วยการสร้าง Dashboard และกราฟจากข้อมูล รองรับการแสวงข้อมูลจาก Time Series Databases 

## IoT Sensor

IoT Sensor เป็นอุปกรณ์ที่ใช้ในการรับค่าต่างๆ แล้วส่งข้อมูลที่ได้ไปยังระบบหรือเครือข่ายอินเทอร์เน็ตเพื่อประมวลผลและแสดงผลแบบเรียลไทม์ ซึ่งเป็นระบบที่อุปกรณ์หลายๆ ชิ้นสามารถเชื่อมต่อกับอินเทอร์เน็ตได้ ทำให้สามารถสื่อสารและส่งข้อมูลระหว่างกันได้อัตโนมัติ