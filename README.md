> :warning: **Work in progress but operational**

# WrtPresence-HA-MQTT

OpenWrt device presence detection bash script. Runs on several APs. Listens "passively" for events from the OpenWrt logread via syslog-ng on a master AP. Can "actively" resynchronise by running "wrtwifistareport" on slave APs every 5 minutes in the event of missed events. Events are transmitted to MQTT. This is then transmitted to Homeassistant via MQTT.
There is no link with knows_device. You find your device in a WrtPresence MQTT device. You can see the attributes of each device, if your device changes access point, it will be updated.
To delete a device, I recommend using MQTT Explorer.

OpenWrt -> mqtt -> homeassitant

## Screenshots

![device](https://raw.githubusercontent.com/dckiller51/WrtPresence-HA-MQTT/master/screenshot/device.png)
![attributes](https://raw.githubusercontent.com/dckiller51/WrtPresence-HA-MQTT/master/screenshot/attributes.png)

Exemple : device_tracker discovery:

topic = `homeassistant/device_tracker/ab12cd23ab12/config` (unique_id = Address mac)

```json
{
  "unique_id": "ab12cd23ab12", (Address mac)
  "name": "Computer HP", (hostname in openwrt if available in dhcp.leases otherwise mac address)
  "device": {
    "manufacturer": "Xiaomi", (Can be modified  in `wrtpresence.conf`)
    "model": "Ax3600", (Can be modified  in `wrtpresence.conf`)
    "name": "WrtPresence", (Can be modified in the MQTT Config node)
    "identifiers": [
      "WrtPresence" (Can be modified in the MQTT Config node)
    ]
  },
  "state_topic": "openwrt/ab12cd23ab12/state", (unique_id = Address mac)
  "payload_home": "home",
  "payload_not_home": "not_home",
  "entity_category": "diagnostic",
  "json_attributes_topic": "openwrt/ab12cd23ab12/attributes" (unique_id = Address mac)
}
```

state = `wrtpresence/ab12cd23ab12/state` (unique_id = Address mac)

```json
home (or not_home)
```

attributes = `wrtpresence/ab12cd23ab12/attributes` (unique_id = Address mac)

```json
{
  "mac": "ab:12:cd:23:ab:12",
  "source_type": "WifiAP-02", (Device name Openwrt WifiAP-01 or WifiAP-02...)
  "source_ssid": "2.4ghz", (Can be modified  in `wrtpresence.conf`)
  "ip": "192.x.x.x" (IP assigned in dhcp.leases)
}
```

## Installation Openwrt

The hostname is important. Apply a simple logic.

```text

WifiAP-01
WifiAP-02
WifiAP-XX
```

### MASTER ROUTER or ACCESS POINT

Copy the 3 files in folder ROOT.

```text
wrtpresence
wrtpresence_main.sh
wrtwifistareport.sh
```

Copy the file in folder /etc/config.
(if you update your router, this avoids reconfiguring everything.)

```text
wrtpresence.conf
```

Change file permissions

```text
chmod +x "/root/wrtpresence"
chmod +x "/root/wrtpresence_main.sh"
chmod +x "/root/wrtwifistareport.sh"
```

Install or remove the following packages

```text
opkg update
opkg install bash
opkg remove logd
opkg install syslog-ng
opkg install jq
opkg install mosquitto-client-nossl
```

Setting syslog-ng.conf line to etc/syslog-ng.conf (example: view in the directory File MASTER ROUTER)

```text
line 68 By default, the line is commented out. You must uncomment it if you have slave access point.
```

Setting wrtpresence.conf. Replace the data on the right.

```text
"mqtt_host":"adresse IP server Mosquitto",
"mqtt_port":"1883",
"mqtt_user":"login",
"mqtt_password":"password",
"phy0_ap0_source_ssid":"IOT",
"phy1_ap0_source_ssid":"5ghz",
"phy2_ap0_source_ssid":"2.4ghz"
"manufacturer":"Xiaomi"
"model":"Ax3600"
```

Optional: 

you can change the ssid name displayed as an attribute.
In this example phy0-ap0 = IOT phy2-ap0 = 2.4ghz and the last is automatically set to 5ghz which corresponds to my phy1-ap0. You must replace only IOT, 2.4ghz and or 5ghz.

You can change the name and model of the device.
In this example, the model is AX3600 and the manufacturer is Xiaomi. Replace AX3600 and Xiaomi by those of your choice.

Add this command line to LUCI / System / Startup / Local Startup

```text
# sh /root/wrtpresence start
```

Setting logging parameters to LUCI / System / System / Logging

```text
External system log server 127.0.0.1
Cron Log Level Warning
```

Add a scheduled task to LUCI / System / Scheduled Tasks

```text
*/5 * * * * /bin/sh "/root/wrtwifistareport.sh" >/dev/null 2>&1
```

Restart Cron:

```text
/etc/init.d/cron restart
```

### SLAVE ACCESS POINT

Copy the file in folder ROOT.

```text
wrtwifistareport.sh
```

Change file permissions

```text
chmod +x "/root/wrtwifistareport.sh"
```

Setting logging parameters to LUCI / System / System / Logging

```text
External system log server [IP ADDRESS OF MASTER ROUTER or ACCESS POINT]
Cron Log Level Warning
```

Add a scheduled task to LUCI / System / Scheduled Tasks

```text
*/5 * * * * /bin/sh "/root/wrtwifistareport.sh" >/dev/null 2>&1
```

Restart Cron:

```text
/etc/init.d/cron restart
```

## Configuration MQTT

You need a mosquitto server. Normally you don't need to change anything in this section.

## Special Thanks

A special thanks you to everyone who helped me with this project in one way or another.

* **Catfriend1** (The author of the bash code) [Github][github]
* **Ed Morton** and **jhnc** (Help with bash code) [Stackoverflow][stackoverflow]

<!-- References -->

[github]: https://github.com/
[stackoverflow]: https://stackoverflow.com/
