# ESP32-C6 Prototyper
A PCB for prototyping using an ESP32-C6-WROOM-1(U) module.

The additional circuitry, board layout, and I/O labelling is oriented towards the development of battery-powered sense/logging, potentially with attachment of a radio module via UART for LoRa, NB-IoT, etc. In particular, use of the ULP co-processor is anticipated.

The main I/O at board edge are arranged to support multiple scenarios:
- standard 0.1" header pins, either straight or right angle for insertion into solderless breadboard, with the single row allowing for plenty of free space to work in.
- standard 0.1" header receptacles for "du pont" wire lash-ups or direct plugging of modules.
- JST-EH (or similar or screw connectors for deployed prototypes).

Most of the components connected to GPIOs are optional. The external crystal (and associated capacitors) is also optional but is anticipated as a normal fitting, given the design use cases and the poor time-keeping of the on-chip RC oscillator, so GPIOs 0 and 1 are not brought out (these "pins" are the crystal connections).

## Supporting Elements
The added elements are largely self-evident from the schematic, but some notes...

The U1/U5 circuit provides for measurement of the voltage of the power source (before LDO reduction), using R20/21 voltage divided to bring the ADC intput to within its Vref range. U1 allows the ADC to be used for another input when not sampling Vin. U5 ensures that there is not a continuous trickle through the potential divider during deep sleep.

Circuits around Q1 and U4 are intended to allow GPIOs to switch Vcc on/off to suppress unwanted peripheral current drain. The load switch (U4) is probably the best option for heavier loads and includes an optional capacitor (C10) if the current surge causes problems. Higher current load switches with the same pinout are available. The circuit around U4 (fed by GPIO22), output to P13 can be directly connected to the UART0 headers via J12. Less demanding loads can use the circuit around Q1, which is driven active-low by GPIO23 and controls output to P10. The An optional resistor in the MCU-MOSFET connection is shorted out; cut the trace and add a resistor if the ESP32 drive circuit and MOSFET gate capacitance cause problems. J18 allows for either temporary bypassing if header pins are fitted, or for P10 to be converted to permanent power out points if a wire link is used.

Pull-down resistors on U4 and U5 could probably be omitted if GPIO15 is set to "hold" the outputs low on deep sleep. Otherwise they are essential to avoid ambient pickup.

Circuits around Q2/3 are intended for driving loads which have a current exceeding the ESP32 output drivers. An optional capacitor is accommodated if the current surge causes problems. J6/7 can be used to use GPIO4/5 as inputs (these GPIOs are available to the ULP so can be used for counting or event-driven task initiation) or as outputs driven direct from the ESP32 (in which case R12/3 can be co-opted as load current control resistors). These  GPIOs are also usable for LP UART, and are the only ones which can, with R12/3 and J6/7 as shorts. R4 and R7 are for load current control and J8/9 are present to allow for quick-change of resistance when breadboarding.

The header marked "CTRL" is intended for a push button to bring the ESP32 out of deep sleep into e.g. offering a WiFi SoftAP for data off-loading, configuration, etc. This is, therefore, a LP GPIO. GPIO15 is intended for conditional signalling while awake and has a series resistor for convenience. These can, of course, be co-opted to generic GPIO use. LP_GPIO3 has a pull-up resistor option for use as a wake-from-sleep push button input.

The GPIO-A1 header may be used for SPI comms (signals routed via GPIO matrix, so not suitable for very fast comms) but also contains all the signal lines for SD card connection, although the PCB traces are rather long for high speed use.

J14 is intended for measuring current consumption.

UART0, brought out as HP_UART, is intended for LoRa or NB-IoT modules, which would be powered from the main board. GPIO22 can be used to switch power off as these are likely to be quite power hungry if they do not have a good sleep mode (and in any case, the main use case has the ESP32 in deep sleep most of the time so even a small quiescent current to such modules is unwelcome. J12 controls whether or not a switched power supply is connected and may also be left compeltely open if a USB-Seriel adapter is connected here (the pinout of P7 is made to suit common adapters) and should not power the board.

J15/16 are intended for attaching removable jumpers, as configuration setting inputs. They are not connected to any GPIO but have an adjacent hole to allow for a flying wire to be easily soldered and any free GPIO used.

## ULP Notes
GPIOs for the ULP to count events or to trigger a wake-up of the HP CPU must use the LP GPIO (0-7), which are marked with an asterisk on the PCB.

I2C (and UART, although that is not specifically brought to a header) capabilities for the ULP are reduced compared to the main CPU.

## Programming and Debugging
Uses the USB CDC and JTAG adapter built into the ESP32-C6. Use of UART0 for stdout logging should be disabled using the following sdkconfig:
```
CONFIG_ESP_CONSOLE_USB_SERIAL_JTAG=y
CONFIG_ESP_CONSOLE_SECONDARY_NONE=y
CONFIG_ESP_CONSOLE_USB_SERIAL_JTAG_ENABLED=y
```

This still leaves some output to UART0 at boot time, which can be suppressed by "strapping" GPIO 15 at boot time or by blowing an eFuse.

The "pins" which can be directly connected to the JTAG controller are xxxxxxxxxxxxxxxxxxxxxxxxxxxx