# Firmware changes for the final board

This lists what the firmware has to change, or at least check, compared with the original Galapagos board. Pins are ESP32 GPIO numbers.

## Changed wiring

MAX31855 data out moved from GPIO12 to GPIO34. GPIO12 is a strapping pin (MTDI selects the flash voltage at boot), so a module holding it high could stop the ESP32 from booting. Clock (GPIO25) and chip select (GPIO15) did not change. GPIO34 is input only and has no internal pull-up or pull-down, which is fine because the MAX31855 drives DO while CS is low. Replace 12 with 34 in the MAX31855 constructor or software SPI setup, and remove any pinMode or pull configuration on 34. The module is now the Adafruit 269 breakout. Its header order only matters to the PCB, and the protocol is the same.

The ESP32 is now powered only from its 3V3 pin, and the 5V pin is not connected. The firmware needs no change, but with only USB plugged in the DevKit regulator feeds the 3.3 V rail and the 5 V rail stays dead, so the SD card does not start. Bench tests that use the SD card need the battery connected. The firmware should treat SD.begin failure as a recoverable state and not hang.

## New connections

GPS over UART was added on a second header (U10), next to the original I2C header (U7). Only one GPS is fitted. GPS TX goes to GPIO35, the ESP32 RX. GPIO32, the ESP32 TX, goes to GPS RX. Use UART2, for example Serial2.begin(9600, SERIAL_8N1, 35, 32), with 9600 baud as the NEO-8M default. GPIO35 has no pull-up, so with no UART GPS fitted the line floats and can produce garbage bytes. Validate the NMEA checksum, or select the interface at build time. A simple runtime choice is to probe I2C address 0x42 at boot, use I2C if the GPS answers, and fall back to UART otherwise. With a u-blox library (SparkFun u-blox GNSS works for both), the change is only begin(Wire) versus begin(Serial2).

RFM95 DIO1 is now wired to GPIO26. It was not connected before. LMIC needs DIO1 for receive timeouts, so its pin map becomes nss 16, rst 27, dio 5 and 26. RadioLib can use it as the second interrupt: Module(16, 5, 27, 26). The other RFM95 pins are unchanged: VSPI with SCK 18, MISO 19, MOSI 23, CS 16, RST 27, DIO0 5.

## Modules that may differ from what the old code expects

The power monitor is an Adafruit INA260, not an INA226. The bus voltage register (0x02, 1.25 mV per bit) and the default address 0x40 are the same, so a plain register read of the battery voltage still works. Libraries that check the die ID will refuse the chip, though (INA226 reads 0x2260, INA260 reads 0x2270), and the INA260 has an internal 2 mOhm shunt with no calibration register. Use the Adafruit_INA260 library, or read register 0x02 directly. The battery is measured by wiring VBUS to the pack off-board. The INA260 accepts up to 36 V, so 12.6 V is fine.

The BME280 is the Adafruit 2652 breakout, which defaults to address 0x77. Many generic modules use 0x76, so check the address in the code, for example bme.begin(0x77).

The IMU is the Adafruit LSM6DSOX 4438 breakout at address 0x6A, with WHO_AM_I 0x6C. Code written for an LSM6DS3 (WHO_AM_I 0x69) will not detect it. Use Adafruit_LSM6DSOX or a library for the LSM6DSO family.

The I2C bus is unchanged: SDA 21, SCL 22. The Adafruit boards each have 10k pull-ups, so 400 kHz works.

## SD card

The SD card stays on HSPI: SCK 14, MOSI 13, MISO 33, CS 17. MISO is on 33, not the native HSPI pin 12, so pass the pins explicitly: SPIClass(HSPI).begin(14, 33, 13, 17). Keep the clock at 20 MHz or below, since MISO goes through the GPIO matrix, which limits input timing to roughly 26 MHz. The new 33 ohm resistor on MISO damps ringing but does not change any firmware.

## Things to set for flight

Put the GPS in Airborne <1g dynamic model (UBX-CFG-NAV5 dynModel 6) at every boot, and save it to battery-backed RAM if the module has a backup cell. In the default portable model the NEO-8M drops the fix above 18 km. This applies to both I2C and UART.

Keep the LoRa bandwidth at 125 kHz or more. The RFM95 crystal drifts in the cold, and at narrower bandwidths the drift can take the signal outside the receiver window.

Log SD writes in small appended blocks and flush often, so a power loss on landing costs only the last block. Use an industrial-temperature card.

IO0, IO2 and IO12 are left free on purpose because they are boot strapping pins. Do not assign them to anything that could be driven at reset.
