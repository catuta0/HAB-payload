# hab-payload

Carrier PCB for a high-altitude balloon payload. An ESP32 DevKitC V4 (WROOM-32) plugs into the board together with breakout modules for GPS (NEO-8M, I2C or UART header, fit only one), INA260 power monitor, BME280, LSM6DSOX IMU, MAX31855 thermocouple amplifier, RFM95 LoRa radio and an SPI SD card. All modules sit on female headers. The ESP32 is powered from 3.3 V only; its 5 V pin is not connected.

Power comes from a 3S Li-ion pack (9 to 12.6 V) on J1, through a 1.5 A PTC, a 13 V TVS and a P-MOSFET reverse polarity switch, into two TPS62160 bucks at 3.3 V and 5 V. Each buck output goes through a pi filter (ferrite, X7R, solid polymer) with the feedback taken before the ferrite, and the GPS, RFM95 and MAX31855 supplies have their own local ferrite. All capacitors are X7R or solid polymer, with no liquid electrolytics, for operation below 0 C.

The fast SPI lines run on the top layer with 33 ohm series termination at the source (ESP32 for SCK and MOSI, SD module for MISO). Ground is poured on both layers and stitched, SMD pads connect solid to the pour. Test points TP1 to TP4 give VBAT, 3V3, 5V and GND.

Open hardware/pcbgalapagos.kicad_pro in KiCad 10. The project symbol library (galapagos.kicad_sym) and footprint library (galapagos_fp.pretty) are registered relative to the project folder. BOM.csv lists every part with its Mouser link (the breakout modules are bought separately), and docs/firmware_changes.md lists what the firmware has to change for this board.
