# iot-home-stack
IoT Home Stack

## Telemetry Service
Home Stack Telemetry Service, which is a microservice that collects data from various sensors by listening MQTT Topics and processes it and transmits command to a sensor by publishing to a MQTT Topic.
- MQTT Subscription Topics
    - `home/alok/telemetry/temperature`
    - `home/alok/telemetry/humidity`
- MQTT Publish Topics
    - `home/alok/device/<<deviceId>>/command`
## Mirco Controllers
### ESP32-GeneralPurpose-1
General purpose ESP32 microcontroller for various sensors and actuators.
- Device ID: `esp32-general-purpose-1`
- MQTT Subscription Topic: `home/alok/device/esp32-general-purpose-1/command`
- MQTT Publish Topic: 
  - `home/alok/telemetry/temperature`
  - `home/alok/telemetry/humidity`
#### Sensors
- DHT11 Temperature and Humidity Sensor
- Light Sensor: Photosensitive Resistor Module
### ESP32-CAM-1
#### Sensors
- Camera Module
- Motion Sensor: HC-SR501 PIR
## MQTT Broker
### Mosquitto on Kubernetes
## MQTT Topics
### Sensor Topics
- `home/alok/telemetry/temperature`
- `home/alok/telemetry/humidity`
### Controller Topics
- `home/alok/device/esp32-general-purpose-1/command`
## MQTT Payloads
### Temperature Payload
```json
{
  "deviceId": "esp32-general-purpose-1",
  "temperature": 25.5,
  "unit": "Celsius"
}
```
### Humidity Payload
```json
{
  "deviceId": "esp32-general-purpose-1",
  "humidity": 60,
  "unit": "%"
}
```
### Command Payload
```json
{
  "deviceId": "esp32-general-purpose-1",
  "command": "turn_on_fan"
}
```
## TLS
### Root Certificate
```shell
openssl genrsa -des3 -out mqtt-signer-ca.key 2048
```
```shell
openssl req -x509 -new -nodes -key mqtt-signer-ca.key -sha256 -days 365 -out mqtt-signer-ca.crt -subj /C=IN/ST=KA/L=Bengaluru/O=Home/CN=alok-signer
```
### Broker Domain Certificate
```shell
openssl genrsa -out server.key 2048
```
```shell
openssl req -new -sha256 -out server.csr -key server.key -subj /C=IN/ST=KA/L=Bengaluru/O=Home/CN=localhost
```
```shell
openssl x509 -req -in server.csr -CA mqtt-signer-ca.crt -CAkey mqtt-signer-ca.key -CAcreateserial -out server.crt -days 360 -sha256
```
### Client Certificate
```shell
openssl genrsa -out mqtt.client.alok.key 2048
```
```shell
openssl req -new -sha256 -key mqtt.client.alok.key -subj /C=IN/ST=KA/L=Bengaluru/O=Home/CN=alok -out mqtt.client.alok.csr
```
```shell
openssl x509 -req -in mqtt.client.alok.csr -CA mqtt-signer-ca.crt -CAkey mqtt-signer-ca.key -CAcreateserial -out mqtt.client.alok.crt -days 365 -sha256
```