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

| docker env variable  | description |
| ------------- | ------------- |
| MQTT_USER     | defines a mosquitto user at the start of the docker container  |
| MQTT_PASSWORD  | defines a mosquitto password at the start of the docker container|


### <a href="https://hub.docker.com/r/wk275/qbusmqtt">wk275/qbusmqtt</a>

### <a href="https://hub.docker.com/r/wk275/qbtools">wk275/qbtools</a>


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

#### Check if your OS is running in 64-bit mode
```
dpkg --print-architecture
```

## System setup

# Remarks
⚠️ wk275/qbtools, wk275/qbmos & wk275/qbusmqtt are not officially supported by Qbus.
