---
title: "Display mobile operator data from RUT955 router in Homeassistant"
date: 2022-03-16T10:00:00+01:00
draft: false
tags: 
  - mqtt
  - homeassistant
  - mobile
---

{{< imgresize "img/ha_screenshot.png" "400x400" "Router connection details in Homeassistant" >}}

The mobile router RUT955 from Teltonika has a MQTT interface which can give information about the mobile operator data like signal strength or current carrier.
Make sure to configure the `MQTT Publisher` with your MQTT broker according to this [manual](https://wiki.teltonika-networks.com/view/RUT955_MQTT#MQTT_Publisher).

## communication model

Unfortunately Teltonika decided to implement a request/response pattern on top of MQTTs publish subscribe pattern in order to retrieve data from the router (see [reference](https://wiki.teltonika-networks.com/view/Monitoring_via_MQTT)).

If you are i.e. interested in the routers signal strength you would need to subscribe to the topic of your router. Notice that we used the wildcard symbol `+` in the second topic level since we do not know the router id yet.

```
mosquitto_sub -h 192.168.1.10 -t router/+/signal -v
```

And then in the next step request the signal data point via a publish message

```
mosquitto_pub -h 192.168.1.10 -t router/get -m signal
```

Example output where you can see the signal strength `-74` and the router id of your device which is `1115790027` in my case:

```
router/1115790027/signal -74
```

## create MQTT sensor entities in Homeassistant

To receive the sensor data from MQTT into Homeassistant the [MQTT Sensor integration](https://www.home-assistant.io/integrations/sensor.mqtt/) can be used. In the following configuration most of the operator data is captured.

```yaml
# configuration.yaml
sensor:
  - platform: mqtt
    name: "Router Operator"
    state_topic: "router/1115790027/operator"
  - platform: mqtt
    name: "Router Network"
    state_topic: "router/1115790027/network"    
  - platform: mqtt
    name: "Router Connection"
    state_topic: "router/1115790027/connection"  
  - platform: mqtt
    name: "Router Wan"
    state_topic: "router/1115790027/wan"
  - platform: mqtt
    name: "Router Uptime"
    state_topic: "router/1115790027/uptime"
  - platform: mqtt
    name: "Router Signal"
    state_topic: "router/1115790027/signal"
    device_class: "signal_strength"
    unit_of_measurement: "dBm"
```

## regularly request data points of interest with an automation

In addition we need to make sure that the data is requested in regular interval. For this a automation with the [time_pattern](https://www.home-assistant.io/docs/automation/trigger/#time-pattern-trigger) trigger and the [mqtt.publish](https://www.home-assistant.io/docs/mqtt/service/) service can be used.

```yaml
# automations.yaml
sensor:
- id: '1647365447073'
  alias: Retrive Router Data From RUT955
  description: 'This automation requests mobile operator data via MQTT every 30 seconds'
  mode: single
  trigger:
  - platform: time_pattern
    seconds: '30'
  action:
  - service: mqtt.publish
    data:
      topic: router/get
      payload: signal
  - service: mqtt.publish
    data:
      topic: router/get
      payload: operator
  - service: mqtt.publish
    data:
      topic: router/get
      payload: network
  - service: mqtt.publish
    data:
      topic: router/get
      payload: connection
  - service: mqtt.publish
    data:
      topic: router/get
      payload: wan
  - service: mqtt.publish
    data:
      topic: router/get
      payload: operator
  - service: mqtt.publish
    data:
      topic: router/get
      payload: uptime
```

## small rant

I can not understand why you would like to implement a request response pattern on top of publish subscribe protocol. Teltonika could have simply published all data points on a regular basis by themselves or use a HTTP Rest API instead of MQTT for this usecase.