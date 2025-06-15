# iot-home-stack
IoT Home Stack

## Telemetry Service
Home Stack Telemetry Service, which is a microservice that collects data from various sensors by listening MQTT Topics and processes it and transmits command to a sensor by publishing to a MQTT Topic.
### MQTT Subscription Topics
  - `home/alok/status/<<deviceId>>`
  - `home/alok/telemetry/temperature/<<deviceId>>`
  - `home/alok/telemetry/humidity/<<deviceId>>`
### MQTT Publish Topics
  - `home/alok/command/<<deviceId>>`
### General MQTT Configurations
- Clean State = False
- QoS = TBD
## Mirco Controllers
### ESP32-GeneralPurpose-1
General purpose ESP32 microcontroller for various sensors and actuators.
- Device ID: `esp32-general-purpose-1`
- MQTT Subscription Topic: 
  - `home/alok/command/esp32-general-purpose-1`
- MQTT Publish Topic: 
  - `home/alok/telemetry/temperature/esp32-general-purpose-1`
  - `home/alok/telemetry/humidity/esp32-general-purpose-1`
#### Sensors
- DHT11 Temperature and Humidity Sensor
- Light Sensor: Photosensitive Resistor Module
### ESP32-CAM-1
#### Sensors
- Camera Module
- Motion Sensor: HC-SR501 PIR
## MQTT Broker
Mosquitto running on Kubernetes
### Miscellaneous Security Configuration
```text
#Possible values are tlsv1.3, tlsv1.2 and tlsv1.1. If left unset, the default allows TLS v1.3 and v1.2
#tls_version

# Path to the PEM encoded server certificate.
certfile /etc/tls/tls.crt
# Path to the PEM encoded keyfile.
keyfile /etc/tls/tls.key

allow_anonymous false

require_certificate true
# cafile and capath define methods of accessing the PEM encoded
# Certificate Authority certificates that will be considered trusted when
# checking incoming client certificates.
# cafile defines the path to a file containing the CA certificates.
cafile /etc/tls/tls.crt

#If require_certificate is true, you may set use_identity_as_username to true to use the CN value from the client certificate as a username. 
#If this is true, the password_file option will not be used for this listener.
#This takes priority over use_subject_as_username if both are set to true.
use_identity_as_username true

acl_file tbd
```
### ACL (Access Control List)
```text
# Allow user "home-telemetry-svc" to subscribe to all topics
user home-telemetry-svc
topic read $SYS/#
topic read #
topic write home/alok/command/esp32-general-purpose-1

# Allow user "esp32-general-purpose-1" to publish to "sensors/temperature" and "sensors/humidity"
user esp32-general-purpose-1
topic write home/alok/status/esp32-general-purpose-1
topic write home/alok/telemetry/temperature/esp32-general-purpose-1
topic write home/alok/telemetry/humidity/esp32-general-purpose-1
topic read home/alok/command/esp32-general-purpose-1
```
## MQTT Topics
### Sensor Topics
- `home/alok/status/esp32-general-purpose-1`
- `home/alok/telemetry/temperature/esp32-general-purpose-1`
- `home/alok/telemetry/humidity/esp32-general-purpose-1`
### Controller Topics
- `home/alok/command/esp32-general-purpose-1`
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
### Device Status Payload
#### Online Status
```json
{
  "deviceId": "esp32-general-purpose-1",
  "status": "online",
  "time": "",
  "ipAddress": "192.168.1.6"
}
```
### Offline Status
```json
{
  "deviceId": "esp32-general-purpose-1",
  "status": "offline"
}
```
### Command Payload
```json
{
  "deviceId": "esp32-general-purpose-1",
  "command": "turn_on_fan"
}
```
## mTLS (Mutual TLS)

### Root Certificate
```shell
openssl genrsa -des3 -out secret/mqtt-signer-ca.key 2048
```
```shell
openssl req -x509 -new -nodes -key secret/mqtt-signer-ca.key -sha256 -days 365 -out secret/mqtt-signer-ca.crt -subj /C=IN/ST=KA/L=Bengaluru/O=Home/CN=alok-signer
```
### Broker Domain Certificate
```shell
openssl genrsa -out secret/server.key 2048
```
```shell
#openssl req -new -sha256 -out secret/server.csr -key secret/server.key -subj /C=IN/ST=KA/L=Bengaluru/O=Home/CN=192.168.1.201
openssl req -new -out secret/server.csr -key secret/server.key -config secret/openssl-domian-csr.conf
```
```shell
openssl x509 -req -in secret/server.csr -CA secret/mqtt-signer-ca.crt -CAkey secret/mqtt-signer-ca.key -CAcreateserial -out secret/server.crt -days 360 -sha256 -copy_extensions copy
```
### Client Certificate - home-telemetry-service
```shell
openssl genrsa -out secret/mqtt.client.home-telemetry-svc.key 2048
```
```shell
openssl req -new -sha256 -key secret/mqtt.client.home-telemetry-svc.key -subj /C=IN/ST=KA/L=Bengaluru/O=Home/CN=home-telemetry-svc -out secret/mqtt.client.home-telemetry-svc.csr
```
```shell
openssl x509 -req -in secret/mqtt.client.home-telemetry-svc.csr -CA secret/mqtt-signer-ca.crt -CAkey secret/mqtt-signer-ca.key -CAcreateserial -out secret/mqtt.client.home-telemetry-svc.crt -days 365 -sha256
```
### Client Certificate - esp32-general-purpose-1
```shell
openssl genrsa -out mqtt.client.esp32-general-purpose-1.key 2048
```
```shell
openssl req -new -sha256 -key mqtt.client.esp32-general-purpose-1.key -subj /C=IN/ST=KA/L=Bengaluru/O=Home/CN=esp32-general-purpose-1 -out mqtt.client.esp32-general-purpose-1.csr
```
```shell
openssl x509 -req -in mqtt.client.esp32-general-purpose-1.csr -CA mqtt-signer-ca.crt -CAkey mqtt-signer-ca.key -CAcreateserial -out mqtt.client.esp32-general-purpose-1.crt -days 365 -sha256
```