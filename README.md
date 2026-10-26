# Juwel-Smart-Feed-2.0---ESP32-hack
Brain transplantatation on this automatic feeder to make it really smart. 

What you will need:
1/ ESP32C3 mini board, or something similar
2/ soldering iron, hot air station recommended
3/ some wires

How to do it:
1/ Unscrew all 4 bolts from the bottom of feeder.
2/ remove PCB board, desolder all the wires from it
3/ remove LCD by pliers or hot air helps a lot, remove main chip (it may be challenge without hotair), remove U1 IC (4,3V LDO)
4/ connect new wire from 3.3V output of the ESP board to original LDO output. see image for better understanding. 
5/ connect new wire from GND of ESP32 to GND of feeder board
6/ connect 5 new wires from buttons to ESP pins 8,9,10,21,22
7/ connect new wire from IR gate LED cathode to ESP32 GPIO 2 
8/ connect new wire from IR gate emitter pull up resistor to ESP GPIO 3
9/ connect new wire from resistor R10 (motor driver) to ESP 1
10/ create new ESPhome device and add this code under captive_portal: line

ESPhome code:

switch:
  - platform: gpio
    pin: GPIO1
    name: "motor"
    id: motor
    restore_mode: ALWAYS_OFF

  - platform: gpio
    pin: GPIO2
    id: ir_led_GND
    internal: true            
    restore_mode: ALWAYS_ON

binary_sensor:
  - platform: gpio
    pin:
      number: GPIO3
      mode:
        input: true
        pullup: false         
      inverted: true
    name: "IR_switch"
    id: ir_switch
    filters:
      - delayed_on: 5ms       
    on_press:
      - lambda: |-
          if (id(krmeni).is_running() && id(otacky_zbyva) > 0) {
            id(otacky_zbyva)--;
          }

  - platform: gpio
    pin:
      number: GPIO8
      mode:
        input: true
        pullup: true
      inverted: true
    name: "Tlacitko"
    id: tl_ha
    filters:
      - delayed_on: 30ms      
      - delayed_off: 30ms

  - platform: gpio
    pin:
      number: GPIO9
      mode:
        input: true
        pullup: true
      inverted: true
    id: tl_1_otacka
    internal: true
    filters:
      - delayed_on: 30ms
      - delayed_off: 30ms
    on_press:
      - script.execute:
          id: krmeni
          otacky: 1

  - platform: gpio
    pin:
      number: GPIO10
      mode:
        input: true
        pullup: true
      inverted: true
    id: tl_2_otacky
    internal: true
    filters:
      - delayed_on: 30ms
      - delayed_off: 30ms
    on_press:
      - script.execute:
          id: krmeni
          otacky: 2

  - platform: gpio
    pin:
      number: GPIO20
      mode:
        input: true
        pullup: true
      inverted: true
    id: tl_3_otacky
    internal: true
    filters:
      - delayed_on: 30ms
      - delayed_off: 30ms
    on_press:
      - script.execute:
          id: krmeni
          otacky: 3

  - platform: gpio
    pin:
      number: GPIO21
      mode:
        input: true
        pullup: true
      inverted: true
    id: tl_4_otacky
    internal: true
    filters:
      - delayed_on: 30ms
      - delayed_off: 30ms
    on_press:
      - script.execute:
          id: krmeni
          otacky: 4

globals:
  - id: otacky_zbyva
    type: int
    initial_value: '0'

number:
  - platform: template
    name: "Pocet otacek"
    id: pocet_otacek
    min_value: 1
    max_value: 5
    step: 1
    initial_value: 1
    optimistic: true
    restore_value: true


button:
  - platform: template
    name: "Krmit"
    on_press:
      - lambda: id(krmeni)->execute((int) id(pocet_otacek).state);

script:
  - id: krmeni
    mode: single              
    parameters:
      otacky: int             
    then:
      - lambda: id(otacky_zbyva) = 0;           
      - switch.turn_off: ir_led_GND            
      - delay: 50ms                            
      - lambda: id(otacky_zbyva) = otacky;
      - switch.turn_on: motor
      - wait_until:
          condition:
            lambda: 'return id(otacky_zbyva) <= 0;'
          timeout: 60s
      - switch.turn_off: motor
      - switch.turn_on: ir_led_GND
