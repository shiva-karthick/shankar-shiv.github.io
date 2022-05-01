---
layout: post
title:  "Introduction to SPI communication"
date:   2019-12-01 16:12:55 +0800
categories: electronics
mathjax: true
---

Note : master/slave is replaced as primary device/secondary device. 

__Contents__
- [Interface](#interface)
    - [SCLK - Clock](#sclk---clock)
    - [MISO(Master In Slave Out) and MOSI(Master Out Slave In)](#misomaster-in-slave-out-and-mosimaster-out-slave-in)
    - [CS(Chip Select)](#cschip-select)
- [Data transmission](#data-transmission)
- [Clock POLarity(CPOL) and Clock PHAse (CPHA)](#clock-polaritycpol-and-clock-phase-cpha)
- [Independent SPI Configuration](#independent-spi-configuration)
- [Dasiy Chain SPI Configuration](#dasiy-chain-spi-configuration)
- [Application of 23LC1024 1Mbit SPI Serial SRAM](#application-of-23lc1024-1mbit-spi-serial-sram)

Serial Peripheral Interface (SPI) is a **synchronous**, **full-duplex** serial data protocol used by microcontrollers for communicating with one or more peripheral devices quickly over short distances. It can also be used for communication between two microcontrollers.

The device that generates the clock signal is called the primary device. SPI devices support much higher clock frequencies compared to I2C interfaces. Users are advised to read the data sheet for the clock frequency specification of the SPI interface.

Serial Peripheral interface is commonly used to talk to a variety of peripherals, such as
+ Sensors : temperature, pressure, ADC
+ Control devices: audio codecs, digital potentiometers, DAC
+ Memory: flash and EEPROM
+ Communications: Ethernet, USB, USART, CAN, IEEE 802.15.4, IEEE 802.11, handheld video games
+ Real-time clocks
+ LCD
+ Any MMC or SD card 
and many more ...

Advantages of SPI vs UART
+ No data framing. There is no overhead with the extra start and stop bits and thus no extra work for the CPU unlike UART.
+ SPI is blazingly fast.
+ Higher throughput than I²C or SMBus. Not limited to any maximum clock speed, enabling potentially high speed
+ No complex hardware require to send and receive the data, only a shift register is required.
+ Complete protocol flexibility for the bits transferred
  + Not limited to 8-bit words
  + Arbitrary choice of message size, content, and purpose
+ Simple software implementation 

Disadvantages of SPI
+ 4 wires are necessary unlike I²C.
+ No error-checking protocol is defined
+ Typically supports only one master device (depends on device's hardware implementation)
+ No acknowledgment from secondary device (the primary device could be transmitting to nowhere and not know it)

## Interface

The standard SPI connection involves a primary microntroller connected to a secondary microcontroller using the Serial CLocK (SCK), Master Out Slave In (MOSI), Master In Slave Out (MISO), and Chip Select (CS) lines.

<img src="/assets/images/Introduction-SPI-communication/SPI-configuration.png" alt="Decision_Making_in_FPGA_Hardware" width="500" height="300"/>

#### SCLK - Clock
The primary microcontroller initiates data transfer using the clock signal. The clock line synchronizes the two communicating devices. For example, this way both can agree that they’ll read and write data on the positive voltage transition. Both primary and secondary devices use the clock signal to synchronize their voltage signals on the MOSI and MISO data lines.

#### MISO(Master In Slave Out) and MOSI(Master Out Slave In)
MOSI and the MISO are the data lines.
With every clock tick, *both* the primary device and the secondary device transmit *and* receive a bit of data on MOSI and MISO lines respectively. As a result, the concept of *send* and *receive* doesn't exist like UART - both devices are sending and receiving all the time.

#### CS(Chip Select)
The primary microcontroller chooses the desired secondary micrcontroller via the Chip Select line.
The CS is active low, which means in order to talk to the secondary device, the primary device needs to send a LOW signal. When multiple secondary devices are used, an individual chip select signal for each secondary device is required from the primary device.

## Data transmission

<img src="/assets/images/Introduction-SPI-communication/SPI_8-bit_circular_transfer.png" alt="Decision_Making_in_FPGA_Hardware" width="500" height="300"/>

The primary device configures the clock, using a frequency supported by the secondary device and selects the secondary device by enabling the CS signal(LOW) in order to begin a transmission. During each SPI clock cycle, a full-duplex data transmission occurs. The data is simultaneously transmitted (shifted out serially onto the MOSI bus) and received (the data on the bus (MISO) is sampled or read in).

<img src="/assets/images/Introduction-SPI-communication/SPI_shift_register.png" alt="Decision_Making_in_FPGA_Hardware" width="500" height="300"/>

Transmissions normally involve two shift registers; one in the primary device and one in the secondary device; they are connected in a **virtual ring topology**. Data is shifted out either with a most significant bit (MSB) or least significant bit(LSB). 

On the clock edge, both the primary device and the secondary device shift out a bit and output it on the transmission line to the counterpart. On the next clock edge, both the primary and secondary devices sample the bit received. After the register bits have been shifted out and in, the primary device and secondary device have exchanged register values. This process repeats until the primary device stops toggling the clock signal, and deselects the secondary device.

The SPI provides the user with flexibility to select the rising or falling edge of the clock (clock polarity) and to select the clock phase. 

## Clock POLarity(CPOL) and Clock PHAse (CPHA)

<img src="/assets/images/Introduction-SPI-communication/timing-diagram.png" alt="timing-diagram" width="500" height="250"/>

The primary device should also configure the clock polarity and phase with respect to the data. 

* CPOL determines the polarity of the clock. The polarities can be converted with a simple inverter if required.
  * CPOL=0 is a clock which idles at 0, and each cycle consists of a pulse of 1. That is, the leading edge is a rising edge, and the trailing edge is a falling edge.
  * CPOL=1 is a clock which idles at 1, and each cycle consists of a pulse of 0. That is, the leading edge is a falling edge, and the trailing edge is a rising edge.

* CPHA determines the timing (i.e. phase) of the data bits relative to the clock pulses. Conversion between these two forms is non-trivial.
  * CPHA = 0. At the first SCLK edge, the __sample__ strobe occurs. At the next SCLK edge, the data is __shifted__. (__Data is sampled on rising edge and shifted out on the falling edge__)
  * CPHA = 1. At the first SCLK edge, the __shifting__ strobe occurs. At the next SCLK edge, the data is __sampled__. (__Data is sampled on the falling edge and shifted out on the rising edge__)

| SPI Mode | CPOL  | CPHA | Clock polarity in idle state |                Clock Phase Used to Sample and/or Shift the Data |
| -------- | :---: | ---: | ---------------------------: | --------------------------------------------------------------: |
| 0        |   0   |    0 |                    Logic LOW | Data sampled on rising edge and shifted out on the falling edge |
| 1        |   0   |    1 |                    Logic LOW |     Data sampled on falling edge and shifted out on rising edge |
| 2        |   1   |    1 |                   Logic HIGH |     Data sampled on falling edge and shifted out on rising edge |
| 3        |   1   |    0 |                   Logic HIGH | Data sampled on rising edge and shifted out on the falling edge |

For example, SPI MODE 0 is; a timing diagram showing clock polarity and phase. Orange lines denote clock leading edges, and blue lines, trailing edges.

<img src="/assets/images/Introduction-SPI-communication/Mode0.png" alt="Mode0" width="500" height="300"/>

## Independent SPI Configuration

<img src="/assets/images/Introduction-SPI-communication/independent-spi.png" alt="independent SPI" width="600" height="400"/>

<img src="/assets/images/Introduction-SPI-communication/MISO-tristate.png" alt="tristate" width="500" height="300"/>

In the independent SPI configuration, an independent chip select line exists for each secondary device. The primary device asserts only one chip select at a time. 

1. Pull-up resistors are used on all chip select signals. When separate software routines initialize each chip select and communicate with its secondary device, pull-up resistors prevent other uninitialized secondary devices from responding.

2. Since the MISO pins of the secondary devices are connected together, they are required to be tri-state pins (high, low or high-impedance), where the high-impedance output must be applied when the secondary device is not selected. When all SPI chips are disabled, the MISO signal should “float” to approximately half the Vcc voltage. If any device is still driving the MISO line, you’ll see a logic high (usually close to 3.3V or 5.0V) or logic low (close to zero volts). This test verifies if the SPI chip is working as intended.
   
3. Protect bus access with SPI.beginTransaction(settings) and SPI.endTransaction().

## Dasiy Chain SPI Configuration

<img src="/assets/images/Introduction-SPI-communication/SPI_three_slaves_daisy_chained.png" alt="dasiy chain" width="500" height="300"/>

The CS line is tied together for all the secondary devices and data propagates from one chip to the next. All chips receive the same SCLK. The MISO of the first chip is connected to the MOSI of the second chip and so on ... 

The whole chain acts as a communication shift register; daisy chaining is often done with shift registers to provide a bank of inputs or outputs through SPI. In this method, as data is propagated from one chip to the next, the number of clock cycles required to transmit data is directly proportional to the chip position in the daisy chain. 

The advantage of the Daisy-Chain method is that you save a chip select signal for each slave SPI device.
Other applications that can potentially interoperate with SPI that require a daisy chain configuration include SGPIO, JTAG and I2C.

## Application of 23LC1024 1Mbit SPI Serial SRAM

<img src="/assets/images/Introduction-SPI-communication/wiring-diagram.png" alt="wiring diagram" width="700" height="700"/>

| Arduino |  23LC1024  |
| ------- | :--------: |
| D13     |    SCLK    |
| D12     |    MISO    |
| D11     |    MOSI    |
| D10     |     CS     |
| 5V      |    VCC     |
| 5V      |    HOLD    |
| 5V      | 10KR -> CS |
| GND     |    VSS     |

```c++

/*
   Used the following components and wire routing:
   (1) Arduino Uno
   (2) Microchip 23LC1024
   (3) 10K Resistor
 */
#include <SPI.h>
 
//SRAM opcodes
#define RDSR        5
#define WRSR        1
#define READ        3
#define WRITE       2
  
//Byte transfer functions
uint8_t Spi23LC1024Read8(uint32_t address) {
  uint8_t read_byte;
 
  PORTB &= ~(1<<PORTB2);        //set SPI_SS low
  SPI.transfer(READ);
  SPI.transfer((uint8_t)(address >> 16) & 0xff);
  SPI.transfer((uint8_t)(address >> 8) & 0xff);
  SPI.transfer((uint8_t)address);
  read_byte = SPI.transfer(0x00);
  PORTB |= (1<<PORTB2);         //set SPI_SS high
  return read_byte;
}
  
void Spi23LC1024Write8(uint32_t address, uint8_t data_byte) {
  PORTB &= ~(1<<PORTB2);        //set SPI_SS low
  SPI.transfer(WRITE);
  SPI.transfer((uint8_t)(address >> 16) & 0xff);
  SPI.transfer((uint8_t)(address >> 8) & 0xff);
  SPI.transfer((uint8_t)address);
  SPI.transfer(data_byte);
  PORTB |= (1<<PORTB2);         //set SPI_SS high
}
  
void setup(void) {
  uint32_t i;
  uint8_t value;
 
  Serial.begin(9600);
  SPI.begin();
 
  for (i=0; i<32; i++) {
    Spi23LC1024Write8(i, (uint8_t)i);
    value = Spi23LC1024Read8(i);
    Serial.println((uint16_t)value);
  }
}
 
void loop() {
}

```


Sources : 
* [https://www.pjrc.com/better-spi-bus-design-in-3-steps/](https://www.pjrc.com/better-spi-bus-design-in-3-steps/)
* [https://en.wikipedia.org/wiki/Serial_Peripheral_Interface#Interface](https://en.wikipedia.org/wiki/Serial_Peripheral_Interface#Interface)
* [https://ucexperiment.wordpress.com/2013/02/26/more-memory-arduino-and-23lc1024/](https://ucexperiment.wordpress.com/2013/02/26/more-memory-arduino-and-23lc1024/)
* [https://www.analog.com/en/analog-dialogue/articles/introduction-to-spi-interface.html#](https://www.analog.com/en/analog-dialogue/articles/introduction-to-spi-interface.html#)


