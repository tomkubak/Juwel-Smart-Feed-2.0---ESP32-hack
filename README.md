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
10/ create new ESPhome device and add attached code under captive_portal: line

