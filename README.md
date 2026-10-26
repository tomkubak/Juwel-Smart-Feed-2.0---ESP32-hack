# Juwel-Smart-Feed-2.0---ESP32-hack


Brain transplantation for an automatic pet feeder to make it really smart. The original control board is replaced with an ESP32-C3 running ESPHome, so the feeder can be controlled from Home Assistant while the physical buttons keep working.

## What you will need

1. ESP32-C3 mini board (or something similar)
2. Soldering iron (hot air station recommended)
3. Some wires

## How to do it

1. Unscrew all 4 bolts from the bottom of the feeder.
2. Remove the PCB and desolder all the wires from it.
3. Remove from the board:
   - the LCD (pliers work, hot air helps a lot),
   - the main chip (may be a challenge without hot air),
   - U1 IC (4.3 V LDO).
4. Connect a wire from the **3.3 V** output of the ESP board to the original **LDO output** pad. See the image below for better understanding.
5. Connect a wire from **GND** of the ESP32 to **GND** of the feeder board.
6. Connect the 5 buttons to ESP pins **GPIO 8, 9, 10, 20, 21**.
7. Connect a wire from the **IR gate LED cathode** to ESP **GPIO 2**.
8. Connect a wire from the **IR gate emitter pull-up resistor** to ESP **GPIO 3**.
9. Connect a wire from resistor **R10** (motor driver) to ESP **GPIO 1**.
10. Create a new ESPHome device and add the code below under the `captive_portal:` line.

<!-- ![Wiring](images/wiring.jpg) -->

## Pinout

| ESP32-C3 pin | Function |
|---|---|
| GPIO 1 | Motor (via R10) |
| GPIO 2 | IR gate LED cathode |
| GPIO 3 | IR gate sensor (emitter pull-up) |
| GPIO 8 | Button – reported to Home Assistant only |
| GPIO 9 | Button – feed 1 rotation |
| GPIO 10 | Button – feed 2 rotations |
| GPIO 20 | Button – feed 3 rotations |
| GPIO 21 | Button – feed 4 rotations |
| 3.3 V | Original LDO output pad |
| GND | Feeder board GND |

## ESPHome code

```yaml
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
```
