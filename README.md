# Blynk-Home-Assistant

This project will provide a Blynk mobile app interface to the Home Assistant home automation application. I developed this app after my family complained that the Home Assistant iOS app was 'too busy.' I have to admit that I agree with them. But, to be fair to the fantastic community at [Home Assistant](http://home-assistant.io) I haven't done much to help myself here. Home Assistant has a tremendous amount of ways to configure the UI to make it not so busy.

The Blynk user interface provides all the right widgets I needed to
create a user interface that was simple, and still had all the controls
needed to interface with all my stuff.

This application as written can control lights (including dimming control),
garage doors, thermostat, but could be adapted to just about any device.

## Getting Started

This project assumes you will be using Platformio to build and program the application. I haven't tried to build it using's Arduino's IDE, but I imagine it would be very simple to adapt it.

## Prerequisites

To build the application as is you'll need to download the project files and install platformio.

1. Download the project files from [Github](https://github.com/ptcryan/Blynk-Home-Assistant).
2. Download and install [Platformio](http://platformio.org)
3. Additionally you'll need an ESP8266 device. I used an [Adafruit Huzzah](https://www.adafruit.com/product/2471), but any of the available ESP8266 devices should work. Except for blinking the onboard LED to indicate operation, there's no HW needs so I/O isn't an issue.
4. Download and install the appropriate [Blynk phone app](http://www.blynk.cc/getting-started/).
5. I'm assuming you already have an operating Home Assistant setup. Getting that setup is both beyond the scope of this document, and very well documented by the Home Assistant community.

## Building and programming

1. With Platformio open the project (from step 1).
2. You'll need to edit the platformio.ini file to set the variable values specific to your environment. In the snippet below from [platformio.ini](https://github.com/ptcryan/Blynk-Home-Assistant/blob/master/platformio.ini) you would replace all the '*'s with your project & setup information.
3. Set the `upload_port` to the serial port your device is connected to, or to the host name/IP address of the device if you're using OTA.
4. The `BLYNK_SERVER_IP` can either be a local IP address, or Blynk's public server. Use the token you receive from the Blynk phone app to put into `BLYNK_AUTH_TOKEN`
5. Save those changes and then compile and upload to your device.

```
[env:huzzah]
platform = espressif8266
board = huzzah
framework = arduino
upload_port = Blynk-Home-Assistant.local
;upload_port = /dev/cu.usbserial-A9XTYIZI
build_flags =
  '-D_WIFI_SSID_="********"'
  '-D_WIFI_PASS_="********"'
  '-D_MQTT_CLIENT_ID_="Blynk-Home-Assistant"'
  '-D_MQTT_SERVER_IP_="***.***.***.***"'
  '-D_MQTT_SERVER_PORT_=1883'
  '-D_MQTT_USER_="********"'
  '-D_MQTT_PASSWORD_="********"'
  '-D_BLYNK_SERVER_IP_="***.***.***.***"'
  '-D_BLYNK_AUTH_TOKEN_="********************************"'
  ```

## Getting Home Assistant to Operate with this Project

### System Diagram

Here's a diagram to help you understand how all the pieces fit together between the phone app, the ESP8266 app, the MQTT server, and Home Assistant Server.

![Diagram](https://github.com/ptcryan/Blynk-Home-Assistant/blob/master/Blynk-Server-Diagram.jpg)

### Using Automations to Send/Receive MQTT Messages

In order for this app to communicate with Home Assistant you'll need to create several automations to both send MQTT messages when certain actions occur, and to respond to actions when the Blynk app sends MQTT messages.

What will be needed is 2 complementary automations for each device that you want to control. In the example I give here for controlling a desk Hue lamp there is one automation for controlling the lamp, and another for reporting the lamp status. The automation for reporting status over MQTT is necessary so that the Blynk app knows the current lamp condition. The other automation is necessary for controlling the lamp from the Blynk app over MQTT.

```
- id: computer_desk_lamp_mqtt_status
  alias: Computer desk lamp mqtt status
  trigger:
    platform: state
    entity_id: light.hue_white_lamp_7
  action:
    service: mqtt.publish
    data:
      topic: "home/desk/status"
      payload_template: '{{ states ("light.hue_white_lamp_7") }}'

- id: computer_desk_lamp_mqtt_control
  alias: Computer desk lamp mqtt control
  trigger:
    platform: mqtt
    topic: "home/desk/set"
  action:
    service_template: 'homeassistant.turn_{{ trigger.payload }}'
    entity_id: light.hue_white_lamp_7
```

You'll need one of these sets of automations for each lamp, switch, sensor, etc. that you will be interfacing to this application.


## Built With

Blynk Home Assistant uses:

[PubSubClient](https://github.com/knolleary/pubsubclient) - Arduino library for pubhlishing & subscribing to MQTT messages.  
[Blynk-library](https://github.com/blynkkk/blynk-library/releases/latest) - Arduino library for interfacing to a Blynk server.  
[Platformio](https://platformio.org) - An open source ecosystem for IoT development

## License

This project is licensed under the MIT License - see the LICENSE file for details


<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.
