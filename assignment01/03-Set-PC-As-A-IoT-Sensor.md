1. install apache Maven into a pc
2. Fix pom.xml file in target folder from u22 branch git to macport branch git (Copy all macport and paste it on to your u22 branch)

this is a ref
https://hub.docker.com/_/maven

https://phoenixnap.com/kb/install-maven-windows


3. use this command in microservice/iot_sensor
docker run --rm --name my-maven-project -v "${PWD}:/usr/src/mymaven" -w /usr/src/mymaven maven:3.8-openjdk-8 mvn clean install

and wait for it to finish

4. docker build the iot file

docker build -t iot-sensor-second .

5. change all the mosquitto:1883 to your ip but don't change the port

6. docker compose up iot-sensor-2 and mosquitto