
# GRGDO1 Secplus GDO Setup Guide

The Secplus GDO implementation by Konnected is an alternative to RATGDO. It features a number of improvements and is significantly more responsive. Secplus GDO automatically detects the security+ version. Event based processing replaces the RATGDO polling and it runs Espressif IDF hardware serial drivers. It's host a clean and easy to use library set decoupling the  secplus and GDO function calls. This implementation only runs on an ESP32 MCU.

This Secplus GDO ESP32 setup guide provides two methods to add the GRGDO1 to an ESPHome/Home Assistant instance. You can add a new device and flash the firmware with a physical device connection via USB serial or you can add it using the pre installed GRGDO1 WIFI Captive portal and upload the firmware via OTA. The WIFI Captive portal method is by far a more convenient method since we only need a cell phone to establish your local WIFI connection. If the method chosen is the captive portal then you can install GRGDO1 physically and proceed to configure it remotely using your phone and ESPHome/Home Assistant after approximately one minute of power on time.

**Important information:**

- NEVER connect to a live AC mains supply while the case is open.
- Do not UART flash the firmware while the GRGDO1 is AC connected, use only the USB to UART adapter that supplies 3.3v DC.
- The ESPhome Home Assistant Add-on is required - see: [ESPhome Getting Started](https://esphome.io/guides/getting_started_hassio.html)
- **If you are updating a current install please clean the build files in HA or in platformio**

## GRGDO1 Device Info

Availble from [https/www.gelidus.ca/](https/www.gelidus.ca/)

Inside the enclosure - v1 board.

<img src="/images/gdo/grgdo1.top.bare.jpg" alt="GRGDO1" style="width: 400px;"/>

- 60 x 60 mm PCB
- 4 Pin ESP programming header (3.3v)
- Powerful ESP32S Dual Core 240Mhz 4MB Flash, BLE and WiFi module!
- PCB layout strictly follows ESP antenna specifications providing excellent WiFi range
- 8 Pin Quick connect
- Flash enable button (power cycle based)
- Powered directly with US AC plug receptacle blades (120 VAC)
- US AC receptacle passthrough plug (120 VAC)
- Follows UL/CSA specifications.
- 1 Meter of 3 conductor 22 AWG bell wire
- 3D printed protective safety enclosure.

Inside the enclosure - v2 board differences.

<img src="/images/gdo/GRGDO1.v2.USB-C.JPG" alt="GRGDO1" style="width: 400px;"/>

- 4 Pin Aux port header (GND, 3.3v GPIO26, GPIO25) GPIO2 unsoldered
- USB-C programing interface.


Protective 3D enclosure (FR ABS)

<img src="/images/gdo/grgdo1.enclosed.jpg" alt="GRGDO1" style="width: 400px;"/>

## GDO Setup Guide - ESPHome Device Preparation

To start we create a unique device name like gdo1 since the preloaded firmware contains it it will speed up the editing later, once created we can edit the config to host the GRGDO1 pin set.
<br></br>

![New Device](/images/gdo/add.device.gdo1.jpg)

<br></br>
Since we are appending to the YAML config we can skip this step. We need to add a full configuration from the content in this guide.
<br></br>

![Installation](/images/add.device.skip.jpg)

<br></br>
The GRGDO1 is powered with an ESP32 as shown we need to set it and skip the next prompt.
<br></br>

![Installation](/images/gdo/add.device.gdo1.esp32..jpg)

<br></br>
Skip this step and note the generated unique key, this must be preserved in the config going forward.
<br></br>

![Installation](/images/gdo/add.device.gdo1.esp32..jpg)

## GDO Setup Guide - YAML Configuration

Now we can edit the newly created device named gdo1, preserving the API key and add the example GRGDO1 code after the captive portal line, as shown. We provide an alternate here to ensure upstream changes are well tested for good code quality.

```yaml
# GRGDO setup guide ver 2025.03.23
external_components:
  - source: github://gelidusresearch/esphome-secplus-gdo
    components: [ secplus_gdo ]
    refresh: 0s

substitutions:
  id_prefix: grgdo1
  friendly_name: "GDO"
  uart_tx_pin: GPIO22           # J4 Pin 1 or 3 Red CTRL
  uart_rx_pin: GPIO21           # J4 Pin 1 or 3 Red CTRL
  input_obst_pin: GPIO23        # J4 Pin 4 Grey OBST - Hardware based obstruction input option
  dry_contact_open_pin: GPIO18  # J4 Pin 6 Green
  dry_contact_close_pin: GPIO19 # J4 Pin 7 Blue
  dry_contact_light_pin: GPIO17 # J4 Pin 8 Orange
  tof_sda_pin: GPIO26           # v1 board=GPIO3 v2(USB-C) board=GPIO26
  tof_scl_pin: GPIO25           # v1 board=GPIO1 v2(USB-C) board=GPIO25
  garage_door_cover_name: Garage Door
  garage_light_name: Garage Light
  garage_openings_name: Garage Openings
  garage_lock_name: Lock
  garage_motion_name: Motion
  garage_obstruction_name: Obstruction
  garage_motor_name: Motor
  garage_button_name: Wall Button
  garage_sync_name: Synced

esp32:
  board: esp32dev
  framework:
    type: esp-idf
    version: recommended

esphome:
  name: gdo1
  friendly_name: gdo1
  #project:
  #  name: konnected.garage-door-gdov2 #Required if using homebridge-ratgdo
  #  version: "1.0"
  comment: "ESP32: Garage Door Opener"
  platformio_options:
    build_flags:
      - -Wl,--wrap=esp_panic_handler
  #    - -DUART_SCLK_DEFAULT=UART_SCLK_APB # Required for esphome version older than 2024.12.0 (IDF 5.1.5)

wifi:
  power_save_mode: none
  on_connect:
    lambda: id(grgdo).start_gdo();
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  ap:
    ssid: "gdo1"
    password: ""
    ap_timeout: 10s

captive_portal:

logger:
  baud_rate: 921600 # Set to 0 for GRGDO1 v1.0 hardware with the VL53L1X sensor

api:
  encryption:
    key: "YOUR API KEY GOES HERE"

ota:
  platform: esphome

#improv_serial:

#esp32_improv:
#   authorizer: none

web_server:
  include_internal: true

status_led:
  pin: GPIO4

secplus_gdo:
  id: grgdo
  input_gdo_pin: ${uart_rx_pin}
  output_gdo_pin: ${uart_tx_pin}
  #input_obst_pin: ${input_obst_pin} # Used to enable physical pin obstruction sensing otherwise usex secplus data
  #tof_sda_pin: ${tof_sda_pin}       # If defined the ToF code will be enabled and requires number: vehicle_parked_threshold
  #tof_scl_pin: ${tof_scl_pin}       # Required for ToF Sensor

light:
  - platform: secplus_gdo
    name: Garage Door Light
    secplus_gdo_id: grgdo
    id: gdo_light

cover:
  - platform: secplus_gdo
    name: Garage Door
    secplus_gdo_id: grgdo
    id: gdo_door

text_sensor:
  - platform: version
    name: ESPHome Version
    hide_timestamp: true

sensor:
  - platform: secplus_gdo
    secplus_gdo_id: grgdo
    id: gdo_openings
    type: openings
    name: "Garage Door Openings"
    unit_of_measurement: "openings"
    icon: mdi:open-in-app
  - platform: wifi_signal # Reports the WiFi signal strength/RSSI in dB
    id: wifi_signal_db
    update_interval: 60s
    entity_category: "diagnostic"
  - platform: copy # Reports the WiFi signal strength in %
    source_id: wifi_signal_db
    name: "WiFi Signal Strength"
    filters:
      - lambda: return min(max(2 * (x + 100.0), 0.0), 100.0);
    unit_of_measurement: "%"
    entity_category: "diagnostic"
    device_class: ""

# Optional Add-on DHT22
#  - platform: dht
#    model: DHT22
#    pin: GPIO3 # v1 board=GPIO3, v2 board=GPIO26
#    temperature:
#      name: "Temperature"
#      accuracy_decimals: 1
#      filters:                                  # °F vs default °C
#       - lambda: return x * (9.0/5.0) + 32.0;
#      unit_of_measurement: "°F"
#    humidity:
#      name: "Humidity"
#      accuracy_decimals: 1
#    update_interval: 60s

lock:
  - platform: secplus_gdo
    id: gdo_lock_remotes
    secplus_gdo_id: grgdo
    name: "Lock remotes"

binary_sensor:
  - platform: secplus_gdo
    name: "Garage Motion Sensor"
    id: gdo_motion
    secplus_gdo_id: grgdo
    device_class: motion
    type: motion
  - platform: secplus_gdo
    name: "Garage Door Obstruction Sensor"
    id: gdo_obst
    secplus_gdo_id: grgdo
    device_class: problem
    type: obstruction
  - platform: secplus_gdo
    name: "Garage Door Motor"
    id: gdo_motor
    secplus_gdo_id: grgdo
    device_class: running
    type: motor
  - platform: secplus_gdo
    name: "Garage Button"
    id: gdo_button
    secplus_gdo_id: grgdo
    entity_category: diagnostic
    type: button
  - platform: secplus_gdo
    name: $garage_sync_name
    id: gdo_synced
    secplus_gdo_id: grgdo
    type: sync
    device_class: connectivity
  # ToF Sensor
  # - platform: secplus_gdo
  #   secplus_gdo_id: grgdo
  #   id: gdo_vehicle_parked
  #   type: vehicle_parked
  #   name: "Vehicle parked"
  # - platform: secplus_gdo
  #   secplus_gdo_id: grgdo
  #   id: gdo_vehicle_arriving
  #   type: vehicle_arriving
  #   name: "Vehicle arriving"
  # - platform: secplus_gdo
  #   secplus_gdo_id: grgdo
  #   id: gdo_vehicle_leaving
  #   type: vehicle_leaving

  - platform: gpio
    id: "${id_prefix}_dry_contact_open"
    pin:
      number: ${dry_contact_open_pin}  #  dry contact for opening door
      inverted: true
      mode:
        input: true
        pullup: true
    name: "Dry contact open"
    entity_category: diagnostic
    on_press:
      - if:
          condition:
            binary_sensor.is_off: ${id_prefix}_dry_contact_close
          then:
            - cover.open: gdo_door
  - platform: gpio
    id: "${id_prefix}_dry_contact_close"
    pin:
      number: ${dry_contact_close_pin}  # dry contact for closing door
      inverted: true
      mode:
        input: true
        pullup: true
    name: "Dry contact close"
    entity_category: diagnostic
    on_press:
      - if:
          condition:
            binary_sensor.is_off: ${id_prefix}_dry_contact_open
          then:
            - cover.close: gdo_door
  - platform: gpio
    id: "${id_prefix}_dry_contact_light"
    pin:
      number: ${dry_contact_light_pin}  # dry contact for triggering light
      inverted: true
      mode:
        input: true
        pullup: true
    name: "Dry contact light"
    entity_category: diagnostic
    on_press:
      then:
        - light.toggle: gdo_light
    disabled_by_default: false

switch:
  - platform: secplus_gdo
    id: gdo_learn
    type: learn
    secplus_gdo_id: grgdo
    name: Learn
    icon: mdi:plus-box
    entity_category: config

  - platform: secplus_gdo
    id: gdo_toggle_only
    type: toggle_only
    secplus_gdo_id: grgdo
    name: Toggle Only
    icon: mdi:plus-box

select:
  - platform: secplus_gdo
    id: gdo_protocol
    secplus_gdo_id: grgdo
    name: protocol
    icon: mdi:settings
    entity_category: config

number:
  - platform: secplus_gdo
    name: Opening duration
    secplus_gdo_id: grgdo
    entity_category: config
    id: gdo_open_duration
    type: open_duration
    unit_of_measurement: "ms"
    internal: true

  - platform: secplus_gdo
    name: Closing duration
    secplus_gdo_id: grgdo
    entity_category: config
    id: gdo_close_duration
    type: close_duration
    unit_of_measurement: "ms"

  - platform: secplus_gdo
    name: Client ID
    secplus_gdo_id: grgdo
    entity_category: config
    id: gdo_client_id
    type: client_id
    mode: box

  - platform: secplus_gdo
    name: Rolling Code
    secplus_gdo_id: grgdo
    entity_category: config
    id: gdo_rolling_code
    type: rolling_code
    mode: box

  # Remidiates wall button brownouts by throttling TX activity
  # 350ms recommened on 880LM units
  - platform: secplus_gdo
    name: Min Command Interval
    secplus_gdo_id: grgdo
    entity_category: config
    id: gdo_min_command_interval
    type: min_command_interval
    mode: box
    unit_of_measurement: "ms"

  # Add Time Till Close (TTC) control, 0s = disabled
  - platform: secplus_gdo
    name: Time Till Close
    secplus_gdo_id: grgdo
    entity_category: config
    id: gdo_time_to_close
    type: time_to_close
    mode: box
    unit_of_measurement: "s"

  # ToF Sensor - Required
  # - platform: secplus_gdo
  # name: "Vehicle Parked Threshold"
  # secplus_gdo_id: grgdo
  # entity_category: config
  # id: gdo_vehicle_parked_threshold
  # type: vehicle_parked_threshold
  # mode: box
  # unit_of_measurement: "cm"

button:
  - platform: restart
    name: Restart
    id: restart_button
    entity_category: config
  - platform: factory_reset
    name: Factory Reset
    entity_category: config
  - platform: template
    name: Reset door timings
    entity_category: config
    on_press:
      - number.set:
          id: gdo_open_duration
          value: 0
      - number.set:
          id: gdo_close_duration
          value: 0
      - button.press:
          id: restart_button
  - platform: template
    name: Re-sync
    entity_category: config
    on_press:
      - number.increment:
          id: gdo_client_id
      - button.press:
          id: restart_button
```

## GDO Setup Guide - Captive Portal Method

With the GRGDO1 already powered up for at least 1 minute use a phone or other capable device connect to the GRGDO1 default preinstalled captive portal AP SSID named gdo1
<br><br>
<img src="/images/gdo/gdo1.captive.portal.jpg" alt="Select" style="width: 400px;"/>
<br><br>
Set your SSID and Password.
<br><br>
<img src="/images/gdo/gdo1.captive.select.jpg" alt="Select" style="width: 400px;"/>
<br><br>

After a minute or so visit the GRGDO1 WEB portal using its mDNS address to verify it's ready. Power cycle it if required.
http://gdo1.local

Prepare the firmware by selecting INSTALL on the top right of the ESPHome gdo1 edit screen.

Select Wireless.

<img src="/images/gdo/gdo1.esphome.install.jpg" alt="Select" style="width: 400px;"/>


GRGDO1 will now come online and you can add it to Home Assistant with the configured API encryption key.

# GDO Setup Guide - Physical USB Serial Method

To enable flash mode on the GRGDO1 you need to depress and hold SW1 then connect your USB to serial adapter to the UART flashing connector as shown here. Once power is applied the button can be released and GRGDO1 will be in flash mode.  (Pre-connecting J1 and then plugging in the USB end is usually easier)

GRGDO1 v1 Board Flashing Header

<img src="/images/gdo/gdo1.flash.header.JPG" alt="Select" style="width: 200px;"/>

Serial TX and RX pins should be crossed e.g.

<pre><code>J1 PM1 :    USB Adapter
TX     -&gt;   RX
RX     &lt;-   TX
3.3v   -    3.5v Max
GND    -    GND</code></pre>

GRGDO1 v2 Board Flash - plug into the USB-C portal

<img src="/images/gdo/GRGDO1.v2.USB-C.JPG" alt="Select" style="width: 200px;"/>

Prepare the firmware by selecting install on the top right of the ESPHome edit screen.

Select Plug into this computer.

<img src="/images/gdo/gdo1.esphome.install.jpg" alt="Select" style="width: 400px;"/>

GRGDO1 will now come online and you can add it to  Home Assistant with the configured key. Note: If you are trying to test your GRGDO1 configuration while powered by the USB UART, you may run into bootloop issues due to tripping the ESP32's brownout detection. You may need to plug the GRGDO1 into AC power to get it to boot.


## Connecting the GRGDO1 Module

J4 Pinouts

<img src="/images/gdo/gdo1.J4.connector.png" alt="Select" style="width: 400px;"/>

Pins 1 or 3 (CTRL) connects to the RED door opener terminal, this is the wired rolling code control signal. It can also connect to the door control button panel. Two terminals are provided to allow for a secondary feed.

Pins 2 or 5 (GND) connect to one of the WHITE door opener terminals, this is the system GND. Two terminals are provided to allow for a secondary feed.

Pins 6,7 and 8 are the dry contact inputs. This is the default config from the YAML file. These inputs can be used for other purposes such as a door status switch. They are voltage clamped to protect the ESP inputs from over voltages. This are not compatible with non-secplus GDO's

## Example Wired Connection
<img src="/images/gdo/gdo1.connection.yellow.jpg" alt="Select" style="width: 400px;"/>

This example is a basic config and is the most common way to connect up the GRGDO1 hardware

## DHT22 Add-On Connections
<img src="/images/gdo/dht22.grgdo1.connection.png" alt="Select" style="width: 400px;"/>

## ToF Sensor Add-on

The ToF sensor is keyed and connects one way by aligning the pins in the Aux port.

<img src="/images/gdo/GRGDO1.ToF.VL53L1X.Add-On.JPG" alt="Select" style="width: 400px;"/>
<img src="/images/gdo/GRGDO1.black.side.JPG" alt="Select" style="width: 400px;"/>



## Home Assistant Dashboard Examples

Button Card

Our GDO setup guide includes this custom button card which provides a compact simple interface for quick ops on a phone. This example assumes you have a HACS frontend installed with button-card and card-mod setup.

Compact Button Card - Closed State

<img src="/images/gdo/GDO.Button.Closed.jpg" alt="Select" style="width: 400px;"/>

Compact Button Card - Transition State

<img src="/images/gdo/GDO.Button.Transition.jpg" alt="Select" style="width: 400px;"/>

Compact Button Card -  Open State

<img src="/images/gdo/GDO.Button.Open.jpg" alt="Select" style="width: 400px;"/>

# HA Button Card YAML Example

```yaml
# Button part
type: custom:button-card
template: garage-door-button-card
entity: cover.gdo1_door
name: Garage Door
tap_action:
  action: toggle
variables:
  icon_open: mdi:garage-open
  icon_closed: mdi:garage
  icon_cached: mdi:cached

# Template part
garage-door-button-card:
  variables:
    - icon_on: ''
    - icon_off: ''
    - icon_locked: ''
  icon: |
    [[[ if (entity.state == "open") return variables.icon_open;
        if (entity.state == "closed") return variables.icon_closed;
         else return variables.icon_cached;
    ]]]
  size: 40px
  action: toggle
  hold_action:
    action: more-info
  styles:
    card:
      - aspect_ratio: 1/1
      - padding: 5px 5px
      - border-radius: 2
      - border-color: var(--primary-color)
    icon:
      - color: |
          [[[ if (entity.state == "closed") return 'var(--state-icon-color)' ;
              else return 'var(--state-icon-active-color)';
          ]]]
```

## Entity Card

In this GDO setup guide example we use the default entity card with a custom card-mod to high light it's border, in this case the custom card-mod is optional.

Entity Card - Closed

<img src="/images/gdo/GDO.Entity.Closed.jpg" alt="Select" style="width: 400px;"/>

Entity Card - Transition

<img src="/images/gdo/GDO.Entity.Open.jpg" alt="Select" style="width: 400px;"/>

Entity Card - Open

<img src="/images/gdo/GDO.Entity.Transition.jpg" alt="Select" style="width: 400px;"/>

## Entities example

```yaml
type: entities
entities:
  - entity: cover.gdo1_door
    name: Garage Door Opener
title: Garage Door
card_mod:
  style: |
    ha-card {
      border-color: var(--primary-color);
    }
```

 This completes the GDO Setup Guide. For addition help please join the ESPhome discord site and ping @descipher

More to see @ https://www.gelidus.ca
