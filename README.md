# hab-payload

Carrier PCB for a high-altitude balloon payload. An ESP32 DevKit plugs into the board together with breakout modules for GPS (NEO), INA226 power monitor, BME280, LSM6DS IMU, MAX31855 thermocouple amplifier, RFM95 LoRa radio and an SPI SD card. All modules sit on female headers.

Power comes from a rechargeable Li-ion pack (2S or 3S, with its own protection BMS, charged off-board) on J1 and feeds two TPS62160 bucks in parallel, one at 3.3 V and one at 5 V. Each buck output goes through a pi filter (ferrite, 22 uF X7R, 100 uF solid polymer) with the feedback taken before the ferrite. The GPS, RFM95 and MAX31855 supplies have their own local ferrite. Every passive is ceramic X7R or solid polymer tantalum, with no liquid electrolytics, so the board is rated for operation below 0 C.

The fast SPI lines run on the top layer only, with no vias and no bottom copper under them, and keep at least 0.75 mm from other tracks. SCK and MOSI of both SPI buses have 33 ohm series termination at the ESP32. Ground is poured on both layers and stitched, and every signal via has a ground via next to it.

The KiCad 10 project is in hardware (open pcbgalapagos.kicad_pro). BOM.csv lists every part with its Mouser link; the breakout modules are bought separately. Rendered previews are in docs.
