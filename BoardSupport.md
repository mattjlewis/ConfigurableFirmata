# Notes on Specific Boards

## RP2040 / Raspberry Pi Pico

First of all add support for RP2040 boards in the Arduino IDE by going to Boards Manager within the Tools menu.
Enter "RP2040" in the search box, select "Arduino Mbed OS RP2040 Boards" and install the latest version (2.3.1 at the time of writing).

ConfigurableFirmata v2.10.1 requires a minor modification to the `Boards.h` file in the `src/utility` folder for the Pico to work with the Firmata libraries.
On macOS by default this file can be found here: `~/Documents/Arduino/libraries/ConfigurableFirmata/src/utility/Boards.h`.

Add the following prior to the `#else` instruction towards the end of this file to enable support for the Raspberry Pi Pico:

```
// Raspberry Pi Pico
// https://datasheets.raspberrypi.org/pico/Pico-R3-A4-Pinout.pdf
#elif defined(TARGET_RP2040) || defined(TARGET_RASPBERRY_PI_PICO)

#include <stdarg.h>

static inline void attachInterrupt(pin_size_t interruptNumber, voidFuncPtr callback, int mode)
{
   attachInterrupt(interruptNumber, callback, (PinStatus) mode);
}

#define TOTAL_ANALOG_PINS       4
#define TOTAL_PINS              30
#define VERSION_BLINK_PIN       LED_BUILTIN
#define IS_PIN_DIGITAL(p)       (((p) >= 0 && (p) < 23) || (p) == LED_BUILTIN)
#define IS_PIN_ANALOG(p)        ((p) >= 26 && (p) < 26 + TOTAL_ANALOG_PINS)
#define IS_PIN_PWM(p)           digitalPinHasPWM(p)
#define IS_PIN_SERVO(p)         (IS_PIN_DIGITAL(p) && (p) != LED_BUILTIN)
// From the data sheet I2C-0 defaults to GP 4 (SDA) & 5 (SCL) (physical pins 6 & 7)
// However, v2.3.1 of mbed_rp2040 defines WIRE_HOWMANY to 1 and uses the non-default GPs 6 & 7:
//#define WIRE_HOWMANY  (1)
//#define PIN_WIRE_SDA            (6u)
//#define PIN_WIRE_SCL            (7u)
#define IS_PIN_I2C(p)           ((p) == PIN_WIRE_SDA || (p) == PIN_WIRE_SCL)
// SPI-0 defaults to GP 16 (RX / MISO), 17 (CSn), 18 (SCK) & 19 (TX / MOSI) (physical pins 21, 22, 24, 25)
#define IS_PIN_SPI(p)           ((p) == PIN_SPI_SCK || (p) == PIN_SPI_MOSI || (p) == PIN_SPI_MISO || (p) == PIN_SPI_SS)
// UART-0 defaults to GP 0 (TX) & 1 (RX)
#define IS_PIN_SERIAL(p)        ((p) == 0 || (p) == 1 || (p) == 4 || (p) == 5 || (p) == 8 || (p) == 9 || (p) == 12 || (p) == 13 || (p) == 16 || (p) == 17)
#define PIN_TO_DIGITAL(p)       (p)
#define PIN_TO_ANALOG(p)        ((p) - 26)
#define PIN_TO_PWM(p)           (p)
#define PIN_TO_SERVO(p)         (p)
```

### I2C

The Raspberry Pi Pico [datasheet](https://datasheets.raspberrypi.org/pico/Pico-R3-A4-Pinout.pdf) states that the
default GPIOs for I2C-0 are GP 4 (SDA-0) and GP 5 (SCL-0) (physical pins 6 & 7 respectively).
However, v2.3.1 of the Mbed OS RP2040 library defines the default Firmata `Wire` implementation to use
I2C-1 on GP 6 (SDA-1) and GP 7 (SCL-1) (physical pins 9 and 10).

The pins that are used for I2C communication can be changed by editing the `variants/RASPBERRY_PI_PICO/pins_arduino.h` file.
When using v2.3.1 on macOS, this file can found in the `~/Library/Arduino15/packages/arduino/hardware/mbed_rp2040/2.3.1/variants/RASPBERRY_PI_PICO` folder.
Simply update the `PIN_WIRE_SDA` and `PIN_WIRE_SCL` values, for example to use the default I2C-0 pins:

```
#define PIN_WIRE_SDA  (4u)
#define PIN_WIRE_SCL  (5u
```

Note that while the Pico supports two I2C buses, Firmata currently only supports interfacing with one I2C bus.
Support for two I2C buses can be enabled within the RP2040 toolchain by setting `WIRE_HOWMANY` to `(2)`, and defining `PIN_WIRE_SDA1`, `PIN_WIRE_SCL1`, `I2C_SDA1` and `I2C_SCL1`:

```
#define WIRE_HOWMANY  (2)
#define PIN_WIRE_SDA  (4u)
#define PIN_WIRE_SCL  (5u)
#define PIN_WIRE_SDA1 (6u)
#define PIN_WIRE_SCL1 (7u)
#define I2C_SDA       (digitalPinToPinName(PIN_WIRE_SDA))
#define I2C_SCL       (digitalPinToPinName(PIN_WIRE_SCL))
#define I2C_SDA1      (digitalPinToPinName(PIN_WIRE_SDA1))
#define I2C_SCL1      (digitalPinToPinName(PIN_WIRE_SCL1))
```
