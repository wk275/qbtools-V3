# qbtools-V3
Interface between QBUS devices, Home Assistant devices, InfluxDb v2/Grafana statistics and HTTP devices.

![](https://img.shields.io/badge/release-v3.0-blue) <br/>                  
![](https://img.shields.io/badge/arch-arm64-yellow)
![](https://img.shields.io/badge/-amd64-yellow)
![](https://img.shields.io/badge/-i386-yellow) <br/>
![](https://img.shields.io/badge/interfaces-qbus_devices-green)
![](https://img.shields.io/badge/-home_assistant_devices-green)
![](https://img.shields.io/badge/-node--red-green)
![](https://img.shields.io/badge/-influxDB_v2/grafana_statistics-green)
![](https://img.shields.io/badge/-http_devices-green) <br/>
![](https://img.shields.io/badge/prerequisites-docker-red)
![](https://img.shields.io/badge/-docker--compose-red)

![image](https://github.com/wk275/qbtools-V3/assets/55239601/12a4894d-7ab4-4881-ab23-3de5541ac820)

## Features 
Qbtools V3 is a collection of 3 docker images.
- qbmos: customized mosquitto server
- qbusmqtt: customized Qbus mqtt gateway for docker. (details qbusmqtt see https://github.com/QbusKoen/QbusMqtt-installer)
- qbtools: interface between Qbus devices, Homeassistant devices, InfluxDB/Grafana statistics, Node-red and http devices 

### <a href="https://hub.docker.com/r/wk275/qbmos">wk275/qbmos</a>
Customized mosquitto server with support for docker environment variables.

Docker-compose.yaml

````
  qbmos:
    image: wk275/qbmos:latest
    environment:
      - TZ=Europe/Brussels                                        ## defines timezone
      - MQTT_USER=appmos                                          ## sets the qbmos mosquitto user at runtime. Replace it with your own user name!
      - MQTT_PASSWORD=KLCOejzpJDpzjc310C0293!çàé                  ## sets the qbmos mosquitto password at runtime. Replace it with your own password!
    container_name: qbmos
    restart: unless-stopped
    ports:
      - "11883:1883"                                               ## defines external port as 11883 and container/internal port as 1883
                                                                   ## this mqtt server will be reachable at <server ip address>:11883
                                                                   ## whithin a docker-compose.yaml file it will be available at
                                                                   ## 0.0.0.0:1883 or at qbmos:1883
    volumes:
      - ./mosquitto/data:/mosquitto/data                           ## persist data to external directory. This is neccesary to keep homassistant definitions and MQTT entries in sync! 
````


### <a href="https://hub.docker.com/r/wk275/qbusmqtt">wk275/qbusmqtt</a>
Customized docker installation of QBUSMQTTGW

| docker env variable  | description |
| ------------- | ------------- |
| MQTT_HOST     | MQTT server ip address. Use 0.0.0.0 for container ip address |
| MQTT_PORT     | MQTT server port |
| MQTT_USER     | MQTT user
| MQTT_PASSWORD | MQTT password|

### <a href="https://hub.docker.com/r/wk275/qbtools">wk275/qbtools</a>
#### features
- interface homeassistant

      - TZ=Europe/Brussels
      - MQTT_HOST=qbmos
      - MQTT_PORT=1883
      - MQTT_USER=appmos                                        ## qbmos mosquitto user
      - MQTT_PASSWORD=NCJDeceoXZBUCBZib28EZD9yxshxzoç2703E      ## qbmos mosquitto password
      - INFLUXDB2_URL=http://influxdbV2:8086
      - INFLUXDB2_ORG=nodered
      - INFLUXDB2_BUCKET=qbus
      - INFLUXDB2_TOKEN=WOL3wRhMHvtK791JM0EPODDpXzIbYZL9hnmQjrOcSB6fVX2kA8O1qgTJ5v-1T2LI3cf3ViJ2G7LVIwLp3hhzfA==



## qbusmqtt: 

Following softwares are installed in docker containers.
- mosquitto (message broker)
- node-red (logic processing & creation of Home assistant entities)
- home-assistant (dashboard)
- influxDBv2 (database on 64bit OS architecture systems)  to store qbus and mqtt statistics
- grafana (charts)
- qbusmqtt (gateway between qbus and mqtt broker

The environment is tested on following systems:
- arm64 - raspberry pi 4B- 4GB memory - 64 bit OS - Debian GNU/Linux 11 (bullseye)
- arm64 - odroid C4 4-GB memory - 64 bit OS - Ubuntu 22.04.3 LTS (jammy)
- amd64 - X64 64 bit OS - Ubuntu 22.04.3 LTS (jammy)

## System setup

# Remarks
⚠️ wk275/qbtools, wk275/qbmos & wk275/qbusmqtt are not officially supported by Qbus.
