# qbtools-V3
Interface between QBUS devices, Home Assistant, InfluxDb v2/Grafana and HTTP devices.

![](https://img.shields.io/badge/release-v3.0-blue)                  

![](https://img.shields.io/badge/arch-arm64-yellow)
![](https://img.shields.io/badge/-amd64-yellow)
![](https://img.shields.io/badge/-i386-yellow)

![](https://img.shields.io/badge/interface-qbus_devices-green)
![](https://img.shields.io/badge/-home_assistant_devices-green)
![](https://img.shields.io/badge/-influxDB_v2/grafana_statistics-green)
![](https://img.shields.io/badge/-http_devices-green)

![image](https://github.com/wk275/qbtools-V3/assets/55239601/a0e55525-3bd6-4f78-9bab-c3cfd865ef1f)


## Features 
Qbtools V3 is a collection of 3 docker images.

### <a href="https://hub.docker.com/r/wk275/qbmos">wk275/qbmos</a>
Customized mosquitto MQTT server.
Supports MQTT_USER and MQTT_PASSWORD environment variables in docker-compose.yaml.

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
