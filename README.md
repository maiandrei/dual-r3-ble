# dual-r3-ble
Use Dual R3 as BLE tracker

dual-r3-ble.yaml

substitutions:
  device_name: dual-r3-ble
  device_id: dual_r3
  friendly_name: DualR3
  #device_ip: #your_ip
  device_description: "DualR3 installed here or there"

# inspired from https://esphome.io/components/sensor/bl0939.html
# and https://www.espthings.io/index.php/2021/03/01/the-new-sonoff-dual-r3-is-here/
# and https://github.com/nagyrobi/home-assistant-configuration-examples/blob/main/esphome/ble-sensors-ethernet.yaml

esphome:
  name: ${device_name}
  platform: ESP32
  board: nodemcu-32s

esp32_ble_tracker:
  scan_parameters: # interval = window to maximize catching chances
    interval: 5s # 
    window: 5s # yes, it works
    active: false

# Enable Home Assistant API
api:
  #reboot_timeout: 15min
  #password: !secret api_pass
  #encryption:
  #key: !secret api_key
  
ota:
  password: !secret ota_pass
    
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_pass
   
# Enable fallback hotspot (captive portal) in case wifi connection fails
#  ap:
#    ssid: ${friendly_name} Fallback Hotspot"
#    password: !secret fall_pass
#captive_portal:

# web_server:
#  port: 80

# Disable logging over serial
logger:
  baud_rate: 0

uart: 
  tx_pin: GPIO25
  rx_pin: GPIO26
  baud_rate: 4800
  parity: NONE 
  stop_bits: 2 

status_led:
  pin: 
    number: GPIO13
    #inverted: true

button:
- platform: safe_mode # boot in this mode to be able to OTA with many BLE sensors
  name: ${friendly_name} Safe Mode Boot
  id: ${device_id}_reset

switch:
  - platform: gpio
    name: ${friendly_name} Switch 1
    pin: 
      number: GPIO27
    id: ${device_id}_relay1
    restore_mode: restore_default_off

  - platform: gpio
    name: ${friendly_name} Switch 2
    pin: 
      number: GPIO14
    id: ${device_id}_relay2
    restore_mode: restore_default_off

binary_sensor:
  - platform: status
    name: ${friendly_name} bridge API

  - platform: gpio
    pin:
      number: GPIO0
      mode: INPUT_PULLUP
      inverted: True
    name: '${friendly_name} Button'
    internal: True
    on_click:
    - min_length: 10ms
      max_length: 200ms
      then:
        - switch.toggle: ${device_id}_relay1
    - min_length: 300ms
      max_length: 2000ms
      then:
        - switch.toggle: ${device_id}_relay2

  - platform: gpio
    pin:
      number: GPIO32
      mode: INPUT_PULLUP
      inverted: True
    name: '${friendly_name} Switch 1'
    on_state:
      then:
      - switch.toggle: ${device_id}_relay1

  - platform: gpio
    pin:
      number: GPIO33
      mode: INPUT_PULLUP
      inverted: True
    name: '${friendly_name} Switch 2'
    on_state:
      then:
      - switch.toggle: ${device_id}_relay2

sensor:
  - platform: bl0939 
    update_interval: 30s 
    voltage: 
      name: '${friendly_name} Voltage' 
    current_1: 
      name: '${friendly_name} Current 1' 
    current_2: 
      name: '${friendly_name} Current 2' 
    active_power_1: 
      name: '${friendly_name} Power 1' 
    active_power_2: 
      name: '${friendly_name} Power 2' 
    energy_1: 
      name: '${friendly_name} Energy 1' 
    energy_2: 
      name: '${friendly_name} Energy 2' 
    energy_total: 
      name: '${friendly_name} Energy Total'

  - platform: template
    name: ${friendly_name} Free Memory
    lambda: return heap_caps_get_free_size(MALLOC_CAP_INTERNAL);
    icon: "mdi:memory"
    entity_category: diagnostic
    state_class: measurement
    unit_of_measurement: "b"
    update_interval: 60s

  - platform: uptime
    name: ${friendly_name} Uptime

# Senzorul 1
  - platform: pvvx_mithermometer
    mac_address: "AA:BB:CC:DD:EE:FF"
    #bindkey: "notneededifweareusingpvvxfirmware"
    temperature:
      name: "01 Temperature"
      filters: 
      - or:
        - throttle_average: 300s
        - delta: 0.2
    humidity:
      name: "01 Humidity"
      filters:
      - or:
        - throttle_average: 300s
        - delta: 2
    battery_level:
      name: "01 Battery Level"
      filters:
      - or:
        - throttle_average: 1800s
        - delta: 5
    battery_voltage:
      name: "01 Battery-Voltage"
      filters:
      - or:
        - throttle_average: 300s
        - delta: 0.01
    signal_strength:
      name: "01 Signal"
      filters:
      - or:
        - throttle_average: 300s
        - delta: 20

# Senzorul 2
  - platform: pvvx_mithermometer
  ...and so on...
