
[![Arduino CI](https://github.com/RobTillaart/MCP_DAC/workflows/Arduino%20CI/badge.svg)](https://github.com/marketplace/actions/arduino_ci)
[![Arduino-lint](https://github.com/RobTillaart/MCP_DAC/actions/workflows/arduino-lint.yml/badge.svg)](https://github.com/RobTillaart/MCP_DAC/actions/workflows/arduino-lint.yml)
[![JSON check](https://github.com/RobTillaart/MCP_DAC/actions/workflows/jsoncheck.yml/badge.svg)](https://github.com/RobTillaart/MCP_DAC/actions/workflows/jsoncheck.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](https://github.com/RobTillaart/MCP_DAC/blob/master/LICENSE)
[![GitHub release](https://img.shields.io/github/release/RobTillaart/MCP_DAC.svg?maxAge=3600)](https://github.com/RobTillaart/MCP_DAC/releases)


# MCP_DAC

Arduino library for Microchip SPI DAC's:  8, 10, 12 bits, 1, 2 channel.


## Description

The MCP_DAC is a library for DAC's from Microchip in the MCP48xx en MCP49xx series.
The library is experimental as it is not tested with all different devices.
Please post an issue if there are problems.


|  Type   | Channels | Bits | MaxValue | Voltage reference |
|:--------|:--------:|:----:|:--------:|:-----------------:|
| MCP4801 |  1       |  8   |   255    | internal 2.048 V  |
| MCP4802 |  2       |  8   |   255    | internal 2.048 V  |
| MCP4811 |  1       |  10  |   1023   | internal 2.048 V  |
| MCP4812 |  2       |  10  |   1023   | internal 2.048 V  |
| MCP4821 |  1       |  12  |   4095   | internal 2.048 V  |
| MCP4822 |  2       |  12  |   4095   | internal 2.048 V  |
| MCP4901 |  1       |  8   |   255    | external          |
| MCP4902 |  2       |  8   |   255    | external          |
| MCP4911 |  1       |  10  |   1023   | external          |
| MCP4912 |  2       |  10  |   1023   | external          |
| MCP4921 |  1       |  12  |   4095   | external          |
| MCP4922 |  2       |  12  |   4095   | external          |


The output voltage of the MCP_DAC depends on the voltage supplied,
which is in the range of 2.7V .. 5.5V. Check datasheet for the details.


## Interface


### Constructor

- **MCP_DAC(uint8_t dataOut = 255, uint8_t clock = 255)** Constructor base class.
  Other devices just use their name as class object e.g. MCP4801 with same parameters.
- **begin(uint8_t select, uint8_t latchPin = 255)** defines the select pin.
The select pin is used for device selection in case of multiple SPI devices.
- **uint8_t channels()** returns the number of channels.
- **uint16_t maxValue()** returns the maximum value that can be set.
This relates to the number of bits, see table above.


### Gain

- **bool setGain(uint8_t gain = 1)** gain is 1 (default) or 2.
- **uint8_t getGain()** returns gain set, default 1.

The analog output cannot go beyond the supply voltage.
So if Vref is connected to 5V, gain=2 will not output 10 Volts.


### Write

- **bool analogWrite(uint16_t value, uint8_t channel = 0)** writes value to channel. The value is limited to maxValue.
Returns false in case of an invalid channel.
- **uint16_t lastValue(uint8_t channel = 0)** returns last written value, default for channel 0 as that works for the single DAC devices.
- **void setPercentage(float percentage)** percentage = 0..100.0%  Wrapper around **analogWrite()**.
- **float getPercentage(uint8_t channel = 0)** returns percentage. Wrapper around **lastValue()**.
- **void fastWriteA(uint16_t value)** faster version to write to channel 0. Does not check flags and does not update **lastValue()**
- **void fastWriteB(uint16_t value)** faster version to write to channel 1. Does not check flags and does not update **lastValue()**
- **bool increment(uint8_t channel = 0)** returns true if channel is incremented, false otherwise.
- **bool decrement(uint8_t channel = 0)** returns true if channel is decremented, false otherwise.

For fastest speed there is an example added **MCP4921_standalone.ino**.
That squeezes the most performance out of it for now.
Code for the other MCP4xxx can be written in same way.


### Shutdown

- **void shutDown()** shuts down the device, optional one might need to **triggerLatch()**
- **bool isActive()** returns false if device is in shutdown mode.
Note: **write()** will set active to true again.


### Hardware SPI

To be used only if one needs a specific speed.

- **void setSPIspeed(uint32_t speed)** set SPI transfer rate
- **uint32_t getSPIspeed()** returns SPI transfer rate


### LDAC

- **void setLatchPin(uint8_t latchPin)** defines the latchPin, this is optional. The latchPin is used for simultaneous setting a value in multiple devices. Note the latchPin must be the same for all instances that need to be triggered together.
- **triggerLatch()** toggles the defined latchPin, and all devices that are connected to it.


### Buffered

MCP49xxx series only, see page 20 ==> not functional for MCP48xx series.

- **void setBufferedMode(bool mode = false)** set buffered mode on/off. The default = false, unbuffered.
- **bool getBufferedMode()** returns set value


### Debug

- **void reset()** resets internal variables to initial value. (use with care!)
- **bool usesHWSPI()** returns true if HW SPI is used.


## ESP32 specific

### SPI port selection

This functionality is new in 0.1.2 and it is expected that the interface will change
in the future.

- **void selectHSPI()** in case hardware SPI, the ESP32 has two options HSPI and VSPI.
- **void selectVSPI()** see above.
- **bool usesHSPI()** returns true if HSPI is used.
- **bool usesVSPI()** returns true if VSPI is used.

The **selectVSPI()** or the **selectHSPI()** needs to be called
BEFORE the **begin()** function.


### Experimental

- **void setGPIOpins(uint8_t clk, uint8_t miso, uint8_t mosi, uint8_t select)** overrule GPIO pins of ESP32 for hardware SPI. needs to be called
AFTER the **begin()** function.

```cpp
void setup()
{
  MCP.selectVSPI();
  MCP.begin(15);
  MCP.setGPIOpins(CLK, MISO, MOSI, SELECT);  // SELECT should match the param of begin()
}
```

This interface can change in the future as the **select** pin is known
in the code.


### ESP32 connections to MCP4922 (example)

ESP32 has **four** SPI peripherals from which two can be used.

SPI0 and SPI1 are used to access flash memory. SPI2 and SPI3 are "user" SPI controllers a.k.a. HSPI and VSPI.


| MCP4922  |  HSPI = SPI2  |  VSPI = SPI3  |
|:--------:|:-------------:|:-------------:|
|  CS      |  SELECT = 15  |  SELECT = 5   |
|  SCK     |  SCLK   = 14  |  SCLK   = 18  |
|  SDI     |  MOSI   = 13  |  MOSI   = 23  |
| not used |  MISO   = 12  |  MISO   = 19  |

By using different **SELECT** pins multiple DAC's can be controlled over
one SPI bus.


## RP2040 specific

### SPI port selection

Select the SPI bus on which the device is on. Both need to be called before the **begin()** function. If the function is called after the **begin()** function, changes will only apply if the **end()** and then the **begin()** functions are called.

- **void selectSPI()** Select the SPI (SPI0) bus. This is the default and  does not need to be called.
- **void selectSPI1()** Select the (second) SPI1 bus.
- **bool usesSPI()** returns true if SPI is used
- **bool usesSPI1()** returns true if SPI1 is used


### Experimental

- **void setGPIOpins(uint8_t clk, uint8_t miso, uint8_t mosi, uint8_t select)** overrule GPIO pins of RP2040 for different SPI pins. needs to be called
AFTER the **begin()** function. Selcted pins must match the RP2040 pinout!


```cpp
void setup()
{
  MCP.selectSPI();    //use for SPI / SPI0
  MCP.selectSPI1();   //use for SPI1
  MCP.begin(17);
  MCP.setGPIOpins(CLK, MISO, MOSI, SELECT);  // SELECT should match the param of begin()
}
```

### Pico connections to MCP4922 (example)

The RP2040 has **two** SPI peripherals from which two can be used.

SPI (SPI0) and SPI1 can both be used to connect devices.


| MCP4922  |   SPI / SPI0  |      SPI1     |
|:--------:|:-------------:|:-------------:|
|  CS      |  SELECT = 17  |  SELECT = 13  |
|  SCK     |  SCLK   = 18  |  SCLK   = 14  |
|  SDI     |  MOSI   = 19  |  MOSI   = 15  |
| not used |  MISO   = 16  |  MISO   = 12  |

By using different **SELECT** pins multiple DAC's can be controlled over
one SPI bus.


## Operation

See examples


## Future

#### Must

- test test test and ....
- improve documentation.

#### Should

- **useSPI1** and **useHSPI** are functional identical indicators.
  - how to refactor to a generic model? Should work for all libraries.
  - int8_t HWSPIport = 0, 1, 2, 3, 4, ....  (-1 == SW SPI ?).
    - numbers are not self-documenting.
    - **selectSPIport(int)** ?
  - would reduce conditional code.

#### Could

- refactor the API (how).
- minimize conditional in code if possible.


