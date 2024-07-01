# qbtools-V3
Qbtools V3 is a collection of 3 docker images. Together they deliver an environment for integrating QBUS with Homeassistant, Influxdb & Grafana, Node-red and http devices.

![](https://img.shields.io/badge/release-v3.0-blue)                 
![](https://img.shields.io/badge/arch-arm64-yellow)
![](https://img.shields.io/badge/-amd64-yellow)
![](https://img.shields.io/badge/-i386-yellow) <br/>
![](https://img.shields.io/badge/interfaces-qbus_devices-green)
![](https://img.shields.io/badge/-home_assistant_devices-green)
![](https://img.shields.io/badge/-influxDB_v2/grafana_statistics-green)
![](https://img.shields.io/badge/-node--red-green)
![](https://img.shields.io/badge/-http_devices-green) <br/>
![](https://img.shields.io/badge/prerequisites-qbus-red)
![](https://img.shields.io/badge/-docker-red)
![](https://img.shields.io/badge/-docker--compose-red)

![image](https://github.com/wk275/qbtools-V3/assets/55239601/12a4894d-7ab4-4881-ab23-3de5541ac820)

## Docker images 
- ### <a href="https://hub.docker.com/r/wk275/qbmos">wk275/qbmos</a>
qbmos is a customized mosquitto server. Instead of defining a user and password via the standard mosquitto tools, you can directly define them via the docker-compose.yaml environment variables MQTT_USER and MQTT_PASSWORD.

- ### <a href="https://hub.docker.com/r/wk275/qbusmqtt">wk275/qbusmqtt</a>
qbusmqtt is a docker image for the Qbus mqtt gateway. (for details about the default qbusmqttgw installation see https://github.com/QbusKoen/QbusMqtt-installer)

- ### <a href="https://hub.docker.com/r/wk275/qbtools">wk275/qbtools</a>
qbTools is the interface between Qbus devices, Homeassistant devices, InfluxDB/Grafana statistics, Node-red and http devices like Shelly.

  #### qbtools features:
  - Create Home assistant entities:
    
    Home assistant entities are created automatically for all Qbus outputs via the MQTT server and Home assistant discovery.
    The root topic is homeassistant.
    <br/> Following qbus outputs types and HA entity types are supported.
    
    |Qbus output type|Home Assistant entity type|
    |----------------|--------------------------|
    |Dimmer          | Light|
    |On/Off          | Switch|
    |Shutter         | Cover|
    |Thermostat      | Climate (heating only)|
    |Scene           | Scene|

  - Create extra HA entities via the /HA_parms/HAparms.js file - section qbusHA.entities. You'll find an example file in the HA_parms directory after starting the qbtools container.
    Just rename is to HAparms.js and modify its contents. When saved, the parameters will be picked up by qbtools and the correspondening HA entities will be modified after a short period.

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
       
    - Modify HA intities before creation:
      You can modified almost every HA entity property before it is send to the HA MQTT discovery topic homeassistant.
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
    - In the HAparms sections you can use following operations
      - "[@value]" refers to another parameter,
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
      The qbtools has an integrated HTTP server. It has 2 methods: qbusGet and qbusSet and it's parameters should be specified in a flat object format. 
      <br/>E.g
        default qbus mqtt object:
            {
              "id":"UL106",
              "properties":
                {
                  "currTemp":28
                },
              "type":"event"
            }
        flat object:
            {  "id": "UL180",
                "properties.currtemp" = 28,
               "type":"event
            }
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

## Home assistant customization:
  - Login to Home assistant on http://homeassistant_server_ip_address:58123
  - After setup go to Setup > Devices and services > Add integration
  - Add MQTT with following parameters (be sure to specify MQTT exactly, not MQTT JSON,...)
    - Broker = qbmos
    - Port = 1883
    - User name = appmos                               ### or your user if changed
    - Password =  NCJDeceoXZBUCBZib28EZD9yxshxzoç2703E ### or your password if changed
  - hit send & complete
  - ##### restart all docker containers ! After a while all Qbus entities states will be refreshed.

## InfluxDB v2 customization:
  - Login to INfluxDB on http://<Influxdb server ip address>:58086
  - Hit buttin "get started"
  - Fill in following items
    - username: choose one
    - password: choose one & remember it :-)
    - confirm password: same as password
    - initial organization name: e.g nodered
    - initial bucket name: init_bucket
  - Hit continue
  - go to API tokens and hit generate API TOKEN > All Access API Token
    - choose a name e.g. qbus
    - hit save
    - copy the token via <CNTRL C> and not via button COPT TO CLIPBOARD
    - close window
  - go to buckets
    - hit button create bucket
    - name e.g. qbus
    - hit create 

  - save above parameters organization, bucket and token to the specific environment variables in the docker-compose.yaml file under section qbtools
  - ##### restart all docker containers !




# Remarks
⚠️ wk275/qbtools, wk275/qbmos & wk275/qbusmqtt are not officially supported by Qbus.
