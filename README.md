# POCAN - Power Output CAN Splitter
## Overview
POCAN is an SR5 board that is meant to combine the functionality of SR4's 12V/GND Splitter and CAN Splitter. It primarily draws 24V power from the battery pack and distributes it to other systems inside and outside the Aux Box through the
use of Molex and RJ45 connectors. Along with power distribution, it also splits the main can line into various branches. There's also an option to power Telem board and the horn using the aux battery.

## Protection
### Power Management
POCAN provides overvoltage/undervoltage protection to all of its outputs through the use of an efuse. An efuse is a power management device that offers fast switching in the case of a power failure event. There are also High Side Switches connected to each output to help protect other subsystems in case one of them short-circuits. POCAN will disable whichever system is causing the short by measuring its current draw via the High Side Switch. There's also protection circuitry for the various ICs on the board

### Isolation
POCAN is an isolated board, which means that the STM on the board has a different ground reference from the battery pack. There are 3 different ICs that cross the isolation barrier on POCAN (Digital Isolator, Isolated Amplifier, and 24-5 DCDC Converter). The digital isolator is a chip that takes in digital input on one side and produces the same digital output on the other side. The isolated amplifier is similar to the digital isolator; however, it interfaces with analog signals instead. The 24 to 5 DCDC is just a converter that provides isolation. 

## Sensing
### Physical Implementation
POCAN utilizes the STM32 microcontroller for sensing analog voltage, tracking faults, and interfacing with switches. POCAN measures the voltage being supplied to each output and the current going through that output. The load switches have diagnostic capabilities that allow the voltage and current of the output to be measured. Because there are 20 analog sensing signals (10 outputs * 2 analog signals) and only 5 ADCs on the STM, there has to be a way to switch down to only 5 signals at a time. This is done using 5 multiplexers that only output 1 signal from a group of 2 (2:1 Mux). The STM only needs to specify 2 selector bits to control the ADC signals, where the MSB selects between voltage sensing and current sensing, and the LSB selects which group of 5 signals to connect to the ADCs.

### Calculating Voltage/Current
Due to various implementation challenges, the voltage of the SNS pin on the load switch goes through attenuation and amplification before reaching the ADC. In order to understand the output of the ADC, you have to first understand the path that was taken from the SNS pin to the ADC.
The design review for POCAN provides a more in-depth and detailed process for calculating the current and voltage, but here's the general process for both:  

#### Current Sensing Example:
&nbsp; $I_{OUT} = 8A$  
  
&nbsp; $V_{SNS} = \frac{I_{OUT}}{4600} * 1000 = \frac{8}{4600} * 1000 = 1.7V$ &nbsp; (Load switch circuitry converts the sensed current to a voltage)  
  
&nbsp; $V_{ADC} = V_{SNS} * \frac{3}{7} = 1.7V * \frac{3}{7} = 0.75V$ &nbsp; (Attenuation and Amplification)  
  
&nbsp; ADC Output = $V_{ADC} * \frac{4095}{3.3} = 0.75 * \frac{4095}{3.3} = 924$ &nbsp; (Conversion to digital value)
  
#### Voltage Sensing Example:
&nbsp; $V_{OUT} = 24V$

&nbsp; $V_{SNS} = V_OUT * 0.0867 = 2.08V$ &nbsp; (Load switch circuitry converts the voltage down to a reasonable level)

&nbsp; $V_{ADC} = V{SNS} * \frac{3}{7} = 2.08 * \frac{3}{7} = 0.89V$  &nbsp; (Attenuation and Amplification)

&nbsp; ADC Output = $V_{ADC} * \frac{4095}{3.3} = 0.89 * \frac{4095}{3.3} = 1106$ &nbsp; (Conversion to digital value)  
  
  
The process above shows how to calculate the digital output of the ADC from a known voltage/current, but you can also compute the sensed voltage/current from a known ADC value. Just follow the steps shown above, but in reverse order.
