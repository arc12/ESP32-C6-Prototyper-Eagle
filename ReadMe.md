# ESP32-C6 Prototyper
A PCB for prototyping using an ESP32-C6-WROOM-1(U) module.

The additional circuitry, board layout, and I/O labelling is oriented towards the development of battery-powered sense/logging, potentially with attachment of a radio module via UART for LoRa, NB-IoT, etc. In particular, use of the ULP co-processor is anticipated.

The main I/O at board edge are arranged to support multiple scenarios:
- standard 0.1" header pins, either straight or right angle for insertion into solderless breadboard, with the single row allowing for plenty of free space to work in.
- standard 0.1" header receptacles for "du point" wire lash-ups or direct plugging of modules.
- JST-EH (or similar) connectors for deployed prototypes.

Most of the components connected to GPIOs are optional. The external crystal is also optional but is anticipated as a normal fitting, given the design use cases, so GPIOs 0 and 1 are not brought out (these "pins" are the crystal connections).

## Supporting Elements
The added elements are largely self-evident from the schematic, but some notes...

JP1-3 are intended for attaching removable jumpers, as configuration setting inputs, but might be convenient for off-board outputs.

Circuits around Q1-3 are intended to allow GPIOs to switch Vcc on/off to suppress unwanted peripheral current drain. Q1 is operable from the ULP and has a direct connection to the ADC i/o port (NB that the same GPIO is used for ULP I2C). Q2/3 are operable from the main CPU only ....

## ULP Notes
GPIOs for the ULP to count events or to trigger a wake-up of the HP CPU must use the LP GPIO (0-7). Use of LP I2C prevents use of SPI.

## Programming and Debugging
Uses the USB CDC and JTAG adapter built into the ESP32-C6. Use of UART0 for stdout logging should be disabled using the following sdkconfig:
```
CONFIG_ESP_CONSOLE_USB_SERIAL_JTAG=y
CONFIG_ESP_CONSOLE_SECONDARY_NONE=y
CONFIG_ESP_CONSOLE_USB_SERIAL_JTAG_ENABLED=y
```

This still leaves some output to UART0 at boot time, which can be suppressed by "strapping" GPIO 15 at boot time or by blowing an eFuse.