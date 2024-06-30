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
- ### <a href="https://hub.docker.com/r/wk275/qbmos">wk275/qbmos</a>
Customized mosquitto server with support for docker environment variables.

- ### <a href="https://hub.docker.com/r/wk275/qbusmqtt">wk275/qbusmqtt</a>
Customized Qbus mqtt gateway for docker. (details qbusmqtt see https://github.com/QbusKoen/QbusMqtt-installer)

- ### <a href="https://hub.docker.com/r/wk275/qbtools">wk275/qbtools</a>
Interface between Qbus devices, Homeassistant devices, InfluxDB/Grafana statistics, Node-red and http devices

### How to install:
Open a login session on your server and execute code below.
It will 
- create a directoy qbtools-v3 in your home-directory
- create a docker-compose.yaml file in this directory
- start-up docker containers

##### Please choose your own user names and passwords and modify all MQTT_USER and MQTT_PASSWORD environment variables conform !

````
mkdir ~/qbtools-v3
cd ~/qbtools-v3
cat > docker-compose.yaml << EOF
services:
  qbmos:
    image: wk275/qbmos:latest
    environment:
      - TZ=Europe/Brussels
      - MQTT_USER=appmos                                        ## qbmos mosquitto user
      - MQTT_PASSWORD=NCJDeceoXZBUCBZib28EZD9yxshxzoç2703E      ## qbmos mosquitto password
    container_name: qbmos-V3
    restart: unless-stopped
    ports:
      - "51883:1883"
    volumes:
      - ./mosquitto/data:/mosquitto/data

  qbusmqtt:
    image: wk275/qbusmqtt:latest
    container_name: qbusmqtt-V3
    restart: unless-stopped
    network_mode: host
    environment:
      - TZ=Europe/Brussels
      - MQTT_HOST=0.0.0.0
      - MQTT_PORT=1883
      - MQTT_USER=appmos                                        ## qbmos mosquitto user
      - MQTT_PASSWORD=NCJDeceoXZBUCBZib28EZD9yxshxzoç2703E      ## qbmos mosquitto password

  qbtools:
    image: wk275/qbtools:latest
    container_name: qbtools-V3
    restart: unless-stopped
    environment:
      - TZ=Europe/Brussels
      - MQTT_HOST=qbmos
      - MQTT_PORT=1883
      - MQTT_USER=appmos                                        ## qbmos mosquitto user
      - MQTT_PASSWORD=NCJDeceoXZBUCBZib28EZD9yxshxzoç2703E      ## qbmos mosquitto password
      - INFLUXDB2_URL=http://influxdbV2:8086
      - INFLUXDB2_ORG=                                          ## create an influx organization nodered and copy it
      - INFLUXDB2_BUCKET=                                       ## create an influx bucket and copy it
      - INFLUXDB2_TOKEN=                                        ## generate an influx API token and copy it
    ports:
      - "51881:1880"
    volumes:
      - ./HA_parms:/HA_parms

 homeassistant:
    image: ghcr.io/home-assistant/home-assistant:latest
    container_name: homeassistant-buildx
    restart: unless-stopped
    environment:
      - TZ=Europe/Brussels
    ports:
      - "58123:8123"
    volumes:
      - "./homeassistant/config:/config"
      - "./homeassistant/local:/.local"
<BR/>
## **********************************************************************
## in Home assistant go to Setup > Devices and services > Add integration
## type in MQTT and click on it
## - Broker = qbmos
## - Port = 1883
## - User name = appmos
## - Password =  NCJDeceoXZBUCBZib28EZD9yxshxzoç2703E
## hit Next
## **********************************************************************
</BR>
 influxdbV2:
    depends_on:
      - qbmos
    image: influxdb:latest
    container_name: influxdbV2-buildx
    restart: unless-stopped
    ports:
      - "8086:8086"
    volumes:
      - ./influxdbV2/data:/var/lib/influxdb2
      - ./influxdbV2/etc:/etc/influxdb2
    environment:
      - INFLUXDB_ADMIN="admin"
      - INFLUXDB_ADMIN_PASSWORD="qbinflux@10"
      - INFLUXDB_USER="qb-influx"
      - INFLUXDB_USER_PASSWORD="qbinflux@10"
      - TZ=Europe/Brussels

EOF
docker compose up -d
````
# Remarks
⚠️ wk275/qbtools, wk275/qbmos & wk275/qbusmqtt are not officially supported by Qbus.
