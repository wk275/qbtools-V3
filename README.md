# qbtools-V3
Qbus interface to Home Assistant, Influxdb/Grafana, HTTP
<tag>
## Features 
tooling environment for qbus and mqtt.
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
