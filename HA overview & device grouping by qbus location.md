# HA overview & device grouping by qbus location
![](https://img.shields.io/badge/Qbtools_release-v3.0.1-blue)     

HAparms @variable can contain HA entitity properties but also Qbus output properties.
Below you'll find the HAparms code to group HA overview and HA devices by QBus location 

First upgrade to the latest release:
````
cd ~/qbtools-v3
docker compose rm --stop --force
docker images -a
docker rmi wk275/qbtools
docker system prune <<<y
docker compose up -d
````

Then add following code in the HAparms.js file in directory ~/qbtools-v3/HA_parms as part of the section parmStructure.ha.regexPost
````
const version = "user"
const version_ = version.replace(/\./g, "_");
const parmStructure = {
    "version": version_,
    "ha": {
        "regexPost": [
            // **************************
            // Start of code block to add
            // **************************
            {
                    "name_regex": ".*",                          // Alle entiteiten worden aangepast
                    "attributes":                                // Volgende HA entity eigenschappen worden aangepast
                            {
                                    "device.identifiers": "[@location]",    // Gebruik qbus locatie als identifier. Alle entiteiten worden in het overzicht hierdoor volgens locatie gegroepeerd.
                                    "device.name": "[@location]",           // In HA instelligen/apparaten en diensten/mqtt wordt er eveneens gegroepeerd per locatie
                                    "device.manufacturer": "QBUS",
                                    "device.model": "Qbus"
                            }
            },
            // ************************
            // End of code block to add
            // ************************
        ]
    }
}
global.set("HAparms." + version_, { parmStructure }, "file");
node.log(version + " Haparms loaded " + new Date().toLocaleString("en-GB"));
node.status(version + " HAparms loaded " + new Date().toLocaleString("en-GB"));

return msg;
````
