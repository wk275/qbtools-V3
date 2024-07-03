# qbtools-V3
The way you install V3.0 is different from V2. 
Instead of installing the environment from a tar file with precustomized softwares, Qbtools V3 is a collection of 3 docker images. 
Together they deliver an environment for integrating QBUS with Homeassistant, Influxdb & Grafana, http devices and mqtt devices.

![](https://img.shields.io/badge/release-v3.0-blue)                 
![](https://img.shields.io/badge/arch-arm64-yellow)
![](https://img.shields.io/badge/-armv7-yellow) 
![](https://img.shields.io/badge/-amd64-yellow)
<br/>
![](https://img.shields.io/badge/interfaces_with-qbus_devices-green)
![](https://img.shields.io/badge/-home_assistant_devices-green)
![](https://img.shields.io/badge/-influxDB_v2/grafana_statistics-green)
![](https://img.shields.io/badge/-http_devices-green)
<br/>
![](https://img.shields.io/badge/prerequisites-qbus-red)
![](https://img.shields.io/badge/-docker-red)
![](https://img.shields.io/badge/-docker--compose-red)

![image](https://github.com/wk275/qbtools-V3/assets/55239601/0f7021bb-f8b5-4dcc-b8d8-907f06191826)


## Docker images 
- ### <a href="https://hub.docker.com/r/wk275/qbmos">wk275/qbmos</a>
qbmos is a customized mosquitto server. Instead of defining a user and password via the standard mosquitto tools, you can directly define them via the docker-compose.yaml environment variables MQTT_USER and MQTT_PASSWORD.

- ### <a href="https://hub.docker.com/r/wk275/qbusmqtt">wk275/qbusmqtt</a>
qbusmqtt is a docker image for the Qbus mqtt gateway. (for details about the default qbusmqttgw installation see https://github.com/QbusKoen/QbusMqtt-installer)

- ### <a href="https://hub.docker.com/r/wk275/qbtools">wk275/qbtools</a>
qbTools is the interface between Qbus devices, Homeassistant devices, InfluxDB/Grafana statistics and http/mqtt devices.

  #### qbtools features:
  - Create Home assistant entities:
    
    Home assistant entities are created automatically for Qbus outputs. This process is based upon a MQTT server and Home assistant discovery. Root topic is homeassistant.

    Qbtools rely on a unique qbus output name. So if duplicates, please correct them first in qbus system manager.
    
    <br/> Following qbus outputs types and HA entity types are supported.
    
    |Qbus output type|Home Assistant entity type|
    |----------------|--------------------------|
    |Dimmer          | Light|
    |On/Off          | Switch|
    |Shutter         | Cover|
    |Thermostat      | Climate (heating only)|
    |Scene           | Scene|
    |Gauge           | Sensor|
<br/>

  - Create extra HA entities via the ~/qbtools-v3/HA_parms/HAparms.js file - section qbusHA.entities. You'll find an example file in the HA_parms directory after starting the qbtools container.
    Just rename is to HAparms.js and modify its contents. When saved, the parameters will be picked up by qbtools and the correspondening HA entities will be created after a short period.

    e.g.
      - create a HA binary_sensor for qbus switch (e.g a garage_door security switch)
      - create a HA sensor for a qbus thermostat
      - create a HA thermostat for a virtual Qbus thermostat with a temperature sensor of a non qbus MQTT device
      <br/>For details see definitions below.
        ````
        "qbusHa": {
            "entities": [
                          { "name_regex": "^Virtual_Binary_sensor1$",       // add a HA bynary_sensor for a qbus switch/toggle output with name "Virtual_Binary_sensor1"
                            "attributes":
                                {
                                    "entity_type": "binary_sensor",
                                    "device_class": "garage_door",          // see https://www.home-assistant.io/integrations/binary_sensor/
                                    "payload_on": true,                     // Qbus uses true en false for th switch on and off confition
                                    "payload_off": false
                                },
                          },
                          { "name_regex": "^Virtual_HVAC_Therm$",            // add a HA sensor for a qbus thermostat (3 sensors will be created).
                            "attributes":                                    // One for evry qbus thermostat property.
                              {                                              // Virtual_HVAC_Therm.currRegime, Virtual_HVAC_Therm.currTemp and Virtual_HVAC_Therm.setTemp.
                                  "entity_type": "sensor",
                                  "icon": "mdi:thermometer"                  // set icon initially to mdi:thermometer. This can later be changed in the ha.regexPost section below if necessary
                              },
                          },
                          { "name_regex": "^Virtual_HVAC_Therm_Temp1$",      // create a HA thermostat for a Virtual qbus thermostat and use the temperature sensor of a non qbus device
                            "attributes":
                              {
                                  "entity_type": "climate",
                                  "current_temperature_template": '{{value_json.tmp.value}}',                     // MQTT shellymotion topic property tmp.value
                                  "current_temperature_topic": "shellies/shellymotion/status"                     // MQTT shellymotion topic 
                              }
                          },
                          { "name_regex": "^Virtual_HVAC_Therm_Temp1$",     // create a HA thermostat for a Virtual qbus thermostat and use the temperature sensor of a non qbus device 
                            "attributes":
                              {
                                  "entity_type": "climate",
                                  "current_temperature_template": '{{value}}',                                    // use the value of the MQTT shelly topic
                                  "current_temperature_topic": "shellies/shellyht/sensor/temperature"             // MQTT shelly ht topic
                              }
                          },
                    ],
                }
        ````
       
  - Modify HA entities before creation:
    You can modify almost every HA entity property before it is send to the HA MQTT discovery topic homeassistant.
    This is also done via the HAparms.js file in section ha.regexPost. You'll find an example file in the HA_parms directory after starting the qbtools container.
    Just rename is to HAparms.js and modify its contents. When saved, the parameters will be picked up by qbtools and the correspondening HA entities will be modified after a short period.
     <br/>For details see definitions below.
     ````
       "ha": {
          "regexPost": [
              {
                  "name_regex": "Virtual_HVAC_Therm.currRegime",
                  "attributes":
                  {
                      "icon": "mdi:clipboard-list-outline",
                  }
              },
              {
                  "name_regex": "Virtual_HVAC_Therm.setTemp",
                  "attributes":
                  {
                      "icon": "mdi:home-thermometer",
                  }
              },
              {
                  "name_regex": "Virtual_HVAC_Therm.currTemp",
                  "attributes":
                  {
                      "icon": "mdi:temperature-celsius",
                  }
              },
              {
                  "name_regex": "_Blinds_",
                  "attributes":
                      { "device_class": "shutter" }
              },
              {
                  "name_regex": "_Power_",
                  "attributes":
                      { "icon": "mdi:power-socket-eu" }
              },
              {
                  "name_regex": "shellyplug",
                  "attributes":
                      { "icon": "mdi:power-plug" }
              }
          ]
      }
      ````
- In the HAparms sections you can use following statements
  
  - "[@value]" refers to another parameter,
  - 
       e.g. "name": "[@unique_id]" means: if "name" parameter is not defined, it will get the value of the "unique_id" parameter.

  - "[@value#slice(split character, join character, start offset, end offset)]" refers to another parameter and does some slice processing on it, e.g,
         ```
        {
            "topic": "shellies/shellyplug2/relay",
            "entity_type": "sensor",
            "name": "test_[@topic#slice(/,_,1,3)]_[@entity_type]",
            "==> name": "test_shellyplug2_relay_sensor"
        },
        ```
- HTTP interface:
  
  qbtools has an integrated HTTP server. It has 2 methods: qbusGet and qbusSet and it's parameters should be specified as a flat object format. 
  <br/>E.g
    ````
    default qbus mqtt object:
      {
          "id":"UL106",
          "properties":
            {
              "currTemp":28
            },
          "type":"event"
        }
    ````
    ````
    flat object:
        {  "id": "UL180",
            "properties.currtemp" = 28,
           "type":"event
        }
  ````
  ### qbusGet

    ````
          http://<ipaddress of qbtools server>:51881/qbusGet?topic=Virtual_HVAC_Therm     //(to disstinguish from other installations, in the example docker file all external ports have a prefix 5)
    ````
          ==> response
          {
            "topic": "cloudapp/QBUSMQTTGW/UL1/UL167/state",
            "qos": 1,
            "retain": false,
            "_msgid": "0285c33dcf41b1a7",
            "payload": {
              "id": "UL167",
              "properties.currRegime": "COMFORT",
              "properties.currTemp": 30,
              "properties.setTemp": 22,
              "type": "state"
            }
          }
    

    ### qbusSet
    ````
          http://<ipaddress of qbtoolsserver>:51881/qbusSettopic=Virtual_HVAC_Therm&payload.currRegime=NACHT
    ````
          ===> response
          {
            "topic": "cloudapp/QBUSMQTTGW/UL1/UL167/setState",
            "retain": false,
            "qos": 0,
            "payload": {
              "id": "UL167",
              "type": "state",
              "properties": {
                "currRegime": "NACHT"
              }
            },
            "_msgid": "6c22415b7899182a"
          }
          
## How to install a complete qbtools environment from scratch:
- Open a login session on your server and execute code below.
It will 
  - create a directoy qbtools-v3 in your home-directory
  - create a docker-compose.yaml file in this directory
  - start-up docker containers

- ##### Please choose your own user names and passwords and modify all MQTT_USER and MQTT_PASSWORD environment variables conform !

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
    container_name: qbmos-v3
    restart: unless-stopped
    ports:
      - "51883:1883"
    volumes:
      - ./mosquitto/data:/mosquitto/data

  qbusmqtt:
    image: wk275/qbusmqtt:latest
    container_name: qbusmqtt-v3
    restart: unless-stopped
    network_mode: host
    environment:
      - TZ=Europe/Brussels
      - MQTT_HOST=0.0.0.0
      - MQTT_PORT=51883
      - MQTT_USER=appmos                                        ## qbmos mosquitto user
      - MQTT_PASSWORD=NCJDeceoXZBUCBZib28EZD9yxshxzoç2703E      ## qbmos mosquitto password

  qbtools:
    image: wk275/qbtools:latest
    container_name: qbtools-v3
    restart: unless-stopped
    environment:
      - TZ=Europe/Brussels
      - MQTT_HOST=qbmos
      - MQTT_PORT=1883
      - MQTT_USER=appmos                                        ## qbmos mosquitto user
      - MQTT_PASSWORD=NCJDeceoXZBUCBZib28EZD9yxshxzoç2703E      ## qbmos mosquitto password
      - INFLUXDB2_URL=http://influxdbV2:8086
      - INFLUXDB2_ORG=                                          ## fill in after setup of influxdb
      - INFLUXDB2_BUCKET=                                       ## fill in after setup of influxdb
      - INFLUXDB2_TOKEN=                                        ## fill in after setup of influxdb
    ports:
      - "51881:1880"
    volumes:
      - ./HA_parms:/HA_parms

  homeassistant:
    image: ghcr.io/home-assistant/home-assistant:latest
    container_name: homeassistant-v3
    restart: unless-stopped
    environment:
      - TZ=Europe/Brussels
    ports:
      - "58123:8123"
    volumes:
      - "./homeassistant/config:/config"
      - "./homeassistant/local:/.local"

  influxdbV2:
    depends_on:
      - qbmos
    image: influxdb:latest
    container_name: influxdbV2-v3
    restart: unless-stopped
    ports:
      - "58086:8086"
    volumes:
      - ./influxdbV2/data:/var/lib/influxdb2
      - ./influxdbV2/etc:/etc/influxdb2
    environment:
      - TZ=Europe/Brussels

EOF
docker compose up -d
````
### log files
Just type docker logs "container-id" -f
````
docker logs qbtools-v3 -f
````

### Home assistant customization:
  - Login to Home assistant on http://homeassistant_server_ip_address:58123
  - After setup go to Setup > Devices and services > Add integration
  - Add MQTT with following parameters (be sure to specify MQTT exactly, not MQTT JSON,...)
    - Broker = qbmos
    - Port = 1883
    - User name = appmos                               ### or your user if changed
    - Password =  NCJDeceoXZBUCBZib28EZD9yxshxzoç2703E ### or your password if changed
  - hit send & complete
  <br/> In the overview you should see all your qbus outputs now.
  - ##### restart your home assistant to refresh the qbus status.

### InfluxDB v2 customization:
  - Login to InfluxDB on http://<Influxdb server ip address>:58086
  - Hit buttin "get started"
  - Fill in following items
    - username: choose one
    - password: choose one & remember it :-)
    - confirm password: same as password
    - initial organization name: e.g nodered
    - initial bucket name: init_bucket
  - Hit continue & configure later
  - go to API tokens and hit generate API TOKEN > All Access API Token
    - choose a name e.g. qbus
    - hit save
    - copy the token via < CNTRL C > and not via button COPY TO CLIPBOARD
    - close window
  - go to buckets
    - hit button create bucket
    - name e.g. qbus
    - hit create 

  - copy above parameters: organization, bucket and token to the specific environment variables in the docker-compose.yaml file in the container section qbtools! Be sure that you do not use quotes of a space before or behind the equal sign
  - ##### restart all docker containers ! After the restart everything should be working.
  - ##### Check qbmos, qbusmqtt and qbtools container log files if you have problems in starting up the environment

## If you have already some components installed yourself
- if you want to use your MQTT server.
  <br/> Just copy the docker-compose sections of qbusmqtt and qbtools and modify the MQTT_* environment variables conform your environment setup..
- if you already have a MQTT server and a qbusmqttgw running: copy the docker-compose qbtools section and modify the MQTT_* environment variables conform your environment setup.

## Issues and enhancements:
Please use the issues tab on this git-hub. I will look into it asap.

## How to upgrade
Because all components use a container based approach it will be very easy to upgrade.
- Bring down your containers
  ````
  cd ~/qbtools-v3
  docker compose rm --stop --force
  ````
- Remove qbtools, qbmos and qbusmqtt images
  ````
  docker images -a
  docker rmi wk275/qbtools wk275/qbmos wk275/qbusmqtt
  docker system prune <<<y
  ````
- do not delete the directories under qbtools-v3, but start the containers again 
````
docker compose up -d
````


# Remarks
⚠️ wk275/qbtools, wk275/qbmos & wk275/qbusmqtt are not officially supported by Qbus.
