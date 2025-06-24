# wt32-eth01-rs485-orno504w

lo scopo è creare un lettore dei dati di tensione, cos fi, corrente, assorbimento attivo e reattivo di una utenza 220v.

hardware: 
- ORNO WE-504 Contatore Energia Elettrica Monofase 80A Porta RS-485 1 Modulo DIN
- WT32-ETH01
- Convertitore da USB a TTL CH340G UART da 3,3 V/5 V
- DollaTek Modulo Ttl 5PCS 5V MAX485 / RS485 su Scheda di Sviluppo MCU RS-485


piattaforma di sviluppo ESPHOME e HOMEASSISTANCE



cabling work

ORNO pin 23 --> pin A modulo RS485
ORNO pin 24 --> gnd
ORNO pin 25 --> pin B moduloRS485

RS485 pin RO --> pin RX esp32 gpio5
RS485 pin RE + pin DE --> esp32 gpio2
RS485 pin D1 --> pin TX esp32 gpio17


codice
---------------------------------------------------------------------------------------

esphome:
  name: esp32-energy-meter
  friendly_name: esp32-energy-meter


esp32:
  board: wt32-eth01
  framework:
    type: arduino


# Enable Home Assistant API
api:
  encryption:
    key: 'custom'

ota:
  - platform: esphome
    password: 'custom'


web_server:
  port: 80


ethernet:
  type: LAN8720
  mdc_pin: GPIO23
  mdio_pin: GPIO18
  clk_mode: GPIO0_IN
  phy_addr: 1
  power_pin: GPIO16


# Optional manual IP (comment for DHCP)
  manual_ip:
    static_ip: xxx.xxx.xxx.xxx
    gateway: xxx.xxx.xxx.xxx
    subnet: 255.255.255.0
    dns1: xxx.xxx.xxx.xxx
    dns2: xxx.xxx.xxx.xxx

# Turn off logging because RX/TX pins used for modbus
logger:
  level: NONE
  baud_rate: 0 # off

uart:
  id: mod_bus
  rx_pin: GPIO05
  tx_pin: GPIO17
  baud_rate: 9600
  parity: EVEN
  data_bits: 8
  stop_bits: 2
  
modbus:
  id: modbus1
  flow_control_pin: GPIO2

modbus_controller:
  - id: orno_we_504
    address: 0x1
    modbus_id: modbus1
    setup_priority: -10
    update_interval: 20s


sensor:
  # Voltage
  - platform: modbus_controller
    modbus_controller_id: orno_we_504
    name: "Voltage"
    id: orno_we_504_modbus_voltage
    register_type: holding
    address: 0
    device_class: VOLTAGE
    unit_of_measurement: "V"
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
      - multiply: 0.1
  
  # Current
  - platform: modbus_controller
    modbus_controller_id: orno_we_504
    name: "Current"
    id: orno_we_504_modbus_current
    register_type: holding
    address: 1
    device_class: CURRENT
    unit_of_measurement: "A"
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
      - multiply: 0.1
  
  # Grid frequency
  - platform: modbus_controller
    modbus_controller_id: orno_we_504
    name: "Grid frequency"
    id: orno_we_504_modbus_grid_frequency
    register_type: holding
    address: 2
    device_class: FREQUENCY
    unit_of_measurement: "Hz"
    value_type: U_WORD
    accuracy_decimals: 2
    filters:
      - multiply: 0.1
  
  # Active power
  - platform: modbus_controller
    modbus_controller_id: orno_we_504
    name: "Active power"
    id: orno_we_504_active_power
    register_type: holding
    address: 3
    device_class: POWER
    unit_of_measurement: "W"
    value_type: U_WORD
    accuracy_decimals: 0
  
  # Reactive power
  - platform: modbus_controller
    modbus_controller_id: orno_we_504
    name: "Reactive power"
    id: orno_we_504_reactive_power
    register_type: holding
    address: 4
    device_class: REACTIVE_POWER
    unit_of_measurement: "var"
    value_type: U_WORD
    accuracy_decimals: 0
  
  # Apparent power
  - platform: modbus_controller
    modbus_controller_id: orno_we_504
    name: "Apparent power"
    id: orno_we_504_apparent_power
    register_type: holding
    address: 5
    device_class: APPARENT_POWER
    unit_of_measurement: "VA"
    value_type: U_WORD
    accuracy_decimals: 0
  
  # Power factor
  - platform: modbus_controller
    modbus_controller_id: orno_we_504
    name: "Power factor"
    id: orno_we_504_power_factor
    register_type: holding
    address: 6
    device_class: POWER_FACTOR
    unit_of_measurement: ""
    value_type: U_WORD
    accuracy_decimals: 3
    filters:
      - multiply: 0.001
  
  # Active energy
  - platform: modbus_controller
    modbus_controller_id: orno_we_504
    name: "Active energy"
    id: orno_we_504_power_active_energy
    register_type: holding
    address: 7
    device_class: ENERGY
    state_class: total
    unit_of_measurement: "kWh"
    value_type: U_DWORD
    accuracy_decimals: 2
    filters:
      - multiply: 0.001
  
  # Reactive energy
  - platform: modbus_controller
    modbus_controller_id: orno_we_504
    name: "Reactive energy"
    id: orno_we_504_power_reactive_energy
    register_type: holding
    address: 9
    device_class: ENERGY
    state_class: total
    unit_of_measurement: "kvarh"
    value_type: U_DWORD
    accuracy_decimals: 2
    filters:
      - multiply: 0.001


button:
  - platform: restart
    name: "RIAVVIO"




