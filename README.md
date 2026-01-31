# POCAN - Power Output CAN Splitter
## Overview
POCAN is an SR5 board that is meant to combine the functionality of SR4's 12V/GND Splitter and CAN Splitter. It primarily draws 24V power from the battery pack and distributes it to other systems inside and outside the Aux Box through the
use of Molex and RJ45 connectors. Along with power distribution, it splits the main can line into various branches. There's also an option to power Telem board, Sensor Suite, and the horn using the aux battery.

## Protection
### Power Management
POCAN provides short-circuit protection to all of its outputs through the TPS27SA08-Q1 load switches. The STM on POCAN will disable whichever system is causing the short by measuring its current draw via the load switch. There's also protection and filter circuitry for the various ICs on the board.

### Isolation
POCAN is a galvanically isolated board, which means that the STM on the board has a different ground reference from the battery pack. There are 4 different ICs that cross the isolation barrier on POCAN (Digital Isolator, I2C Isolator, Isolated CAN Transciever, and 24-5 DCDC Converter). The digital isolator is a chip that takes in digital input on one side and produces the same digital output on the other side. The I2C amplifier is similar to the digital isolator; however, it has bidirectional capabilities for the data line. The 24 to 5 DCDC is just a converter that provides isolated power.

## Sensing
### Physical Implementation
POCAN utilizes the STM32 microcontroller and I2C ADCs to sense current and device temperature of each output. This is done through load switches which have diagnostic capabilities that allow the supply voltage, load current, and device temperature of each output to be measured. Because there are 10 analog sensing signals and only 3 I2C buses on the STM, there has to be a way to switch down to only 3 signals at a time. This is done using the target addressing functionality of the ADS1115 devices. There's a left side and a right side of the board which corresponds to the first and second I2C bus on the STM. From there, the 5 signals on either side of the board can be addresssed individually. As for the STM and load switch interface, there're 3 digital signals that control the load switch (Diagnostic Enable, Sense Select, and Output Enable) and 1 digital signal that's read by the STM (Fault Status).
